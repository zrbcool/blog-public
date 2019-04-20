### 背景篇
#### 现状
目前公司的后端架构基本上是微服务的架构模式，所有的入站流量均通过API网关进入到后端服务，API网关起到一个“保护神”的作用，监控着所有进入的请求，并具备防刷，监控接口性能，报警等重要功能，流量通过网关后服务间的调用均为RPC调用。
![](http://pp2egchi0.bkt.clouddn.com/FtqXlQ2CZ6kHd1Rfbf6JRzYxrTIJ)
#### 过渡期
在经历完微服务改造后，虽然微服务架构给我们带来了不少红利，但是也不免引入一些问题： 
- 服务拆分导致机器数增多 
- 为了降低浪费多个服务部署到一台主机的端口管理成本
- 不一致的交付环境，造成的排查问题成本

为了解决上述问题，经过调查，我们将目光锁定容器服务Kubernetes，以解决我们的问题，本篇文章主要关注ingress-controller部分，所以下面的架构主要突出API网关及ingress-controller。
下图是我们的过渡期方案：
通过引入一个内网的SLB来解决ingress-controller的POD做为我们API网关upstream的服务发现的问题。并且可以通过逐步切换接口到SLB上达到渐进式迁移的作用，粒度可以做到接口+百分比级别。

![](http://pp2egchi0.bkt.clouddn.com/FtFETZ7Ngjn7iomBzcTxUN9dk8qc)
#### 终态
全部迁移完成后，所有的机器回收，如下图所示：
![](http://pp2egchi0.bkt.clouddn.com/FkN4eqHNFlqruc8Z7oG_SfKllhgs)
其实两层SLB是会有浪费的，但是由于我们的API网关目前承担着重要的作用，所以必须找到替代方案，我们也尝试调研了用定制ingress-controller，或者Service Mesh的Istio方案来解决，由于当时Istio 1.1还没发布，并且大规模使用的案例太少，所以我们保守的选择了目前的这种折中方案，后续要切换到Istio上也不是很麻烦。
#### 按业务线分组Ingress Controller
![](http://pp2egchi0.bkt.clouddn.com/FnmAS0JoxHAx9PT8PcJghkmUPUUY)
所有的流量都由一组默认的ingress-controller来承担这个方案我们从一开始就是否定的，所以通过部署多组ingress-controller来达到一定的隔离是必要的。最终会是上面图的那个形态。
### 实践篇
#### 如何启用多ingress-controller
查看ingress-controller的启动命令参数，可以找到：
![](http://pp2egchi0.bkt.clouddn.com/Fl4jrnaE1UOSKQSp-yZ7Oa2jynun)
其意思是只关注携带annotation为"kubernetes.io/ingress.class"，并且值与controller启动参数参数--ingress-class的值相同的ingress会被监测到，不符合的会被忽略，这样就做到了多组ingress-controller，Ingress资源定义之间的隔离。
例如：
```
      containers:
      - args:
        - /nginx-ingress-controller
        - --ingress-class=xwz #看这里
        - --configmap=$(POD_NAMESPACE)/xwz-nginx-configuration
        - --tcp-services-configmap=$(POD_NAMESPACE)/xwz-tcp-services
        - --udp-services-configmap=$(POD_NAMESPACE)/xwz-udp-services
        - --annotations-prefix=nginx.ingress.kubernetes.io
        - --publish-service=$(POD_NAMESPACE)/xwz-nginx-ingress-lb
        - --v=2
```
以及
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "xwz" #看这里
  labels:
    app: tap-revision-ing
    wayne-app: tap-revision
    wayne-ns: coohua
  name: tap-revision-ing
  namespace: coohua
spec:
  rules:
  - host: staging.coohua.com
    http:
      paths:
      - backend:
          serviceName: tap-revision-stable
          servicePort: 80
        path: /

```
接下来我们来看下nginx-ingress在k8s内部署的资源结构，如图所示：
（画图太费时间，后面的图可能手绘的会多一点，请见谅）
![](http://pp2egchi0.bkt.clouddn.com/Fua5Ll6Ug-dx2qGCHuCIQ72HrfOF)
可以看到一组nginx-ingress-controller其实由以下几部分组成：
- ServiceAccount，ClusterRole，ClusterRoleBinding：权限RBAC定义
- Deployment：控制controller的部署，并依赖ServiceAccount，Configmap，Service
- ConfigMap：三个configmap都是保存自定义的controller配置用的
- Service：这里使用type为LoadBalancer的svc主要是利用阿里云基础服务实现的自动绑定到SLB实例的功能

我们的场景是不需要特殊的权限配置的，所以我们就简单的把红框里面的几个资源复制一份修改下其中的几个配置就完成了一组全新ingress-controller的部署，并且与Kubernetes自带的ingress-controller是互相隔离的。
这里我把我写好的配置放到我的github上，读者可以参考：[ingress resources](https://github.com/zrbcool/blog-public/tree/master/k8s%20ingress%20resources) 

创建好新ingress-controller后，如果发现ingress-controller有如下错误
```
E0416 11:31:50.831279       6 leaderelection.go:304] Failed to update lock: configmaps "ingress-controller-leader-xwz" is forbidden: User "system:serviceaccount:kube-system:nginx-ingress-controller" cannot update resource "configmaps" in API group "" in the namespace "kube-system"
```
参考[issue](https://github.com/kubeapps/kubeapps/issues/120)，需要修改clusterrole：nginx-ingress-controller，增加如下内容
```
...
- apiGroups:
  - ""
  resourceNames:
  - ingress-controller-leader-nginx
  - ingress-controller-leader-xwz #将新增加的configmap增加进来，不然会报上面提到的错误
  resources:
  - configmaps
...
```
#### 新增加的ingress-controller如何自动添加到SLB的后端服务列表中
##### 方式一
Service的spec.externalTrafficPolicy当为Cluster的时候：
集群当中的每台主机都可以充当三层路由器，起到负载均衡及转发的作用，但是由于其对请求包进行了SNAT操作，如图所示：
![](http://pp2egchi0.bkt.clouddn.com/FlntGq0Dd9CM1oGy4vr49qwbONRw)

这样导致ingress-controller的POD内获取到的client-ip会是转发包过来的那个worker节点的IP，就会导致我们无法获取客户端的真实IP，所以如果我们不关心客户端真实IP的情况，可以使用这种方式，然后将所有的worker节点IP加入到SLB的后端服务列表当中即可
##### 方式二
Service的spec.externalTrafficPolicy当为Local的时候：
节点只会把请求转给节点内的ingress-controller的POD，不经过SNAT操作ingress controller可以获取到客户端的真实IP，如果节点没有POD，就会报错。这样我们就需要手工维护POD，节点，与SLB后端服务之间的关系。那么有没有一种方式可以自动管理维护这个关系呢？其实是有的，阿里云容器服务为我们做好了这一切，只要在type为LoadBalancer的Service上增加如下几个annotation，它就可以为我们将启动了POD的worker节点的端口及IP自动添加到SLB后端服务当中，扩容缩绒自动更新，如下所示（注意我们使用的是内网SLB，type为intranet，大家根据实际情况修改）：
```
metadata:
  annotations:
    service.beta.kubernetes.io/alicloud-loadbalancer-address-type: intranet
    service.beta.kubernetes.io/alicloud-loadbalancer-force-override-listeners: "true"
    service.beta.kubernetes.io/alicloud-loadbalancer-id: lb-2zec8x×××××××××965vt
```
### 填坑篇
#### nginx worker进程数的问题
我们直到nginx的默认配置worker_processes为auto的时候，会根据当前主机cpu信息自动计算，但是nginx并不是一个cgroups aware的应用，所以其会盲目“自大”的认为有“好多”cpu可以用，这里我们就需要对其进行指定，可以在configmap中设置参数：
```
apiVersion: v1
data:
  worker-processes: "8"
kind: ConfigMap
metadata:
  annotations:
  labels:
    app: ingress-nginx
  name: xwz-nginx-configuration
  namespace: kube-system

```
#### 内核参数设置
这块我们暂时使用默认的deployment当中给定的参数，后续调优的时候会根据情况调整
```
      initContainers:
      - command:
        - /bin/sh
        - -c
        - |
          sysctl -w net.core.somaxconn=65535
          sysctl -w net.ipv4.ip_local_port_range="1024 65535"
          sysctl -w fs.file-max=1048576
          sysctl -w fs.inotify.max_user_instances=16384
          sysctl -w fs.inotify.max_user_watches=524288
          sysctl -w fs.inotify.max_queued_events=16384
```
#### 客户端真实IP问题
小流量灰度期间业务同学反馈说第三方的反作弊发现我们调用他们的接口异常，一轮分析下来发现，原来是发给第三方请求中携带的客户端IP被写为了我们OpenResty的主机IP 
还记得前面的架构图吗？
![](http://pp2egchi0.bkt.clouddn.com/FtFETZ7Ngjn7iomBzcTxUN9dk8qc)
出现这个问题的原因就是我们的ingress controller被放到了OpenResty后面，查看到ingress controller的template中对X-REAL-IP设置的代码部分如下：
![](http://pp2egchi0.bkt.clouddn.com/Fk2yvF82fpL6_KP3fkOErj5oZJ5X)
这个the_real_ip又是哪来的呢？
![](http://pp2egchi0.bkt.clouddn.com/Fqle-PiPDR9AZNmgpUDwWB-qZ2oI) 
可以看到默认情况下，是从remote_addr这个变量获取，在我们的场景下remote_addr就会是OpenResty的主机IP，所以我们需要修改其为OpenResty为我们设置好的X_REAL_IP的header。我们的做法是将template文件导出修改并以congfigmap的方式挂载进镜像，覆盖原来的配置，配置如下：
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  ...
spec:
  template:
    spec:
      containers:
        name: nginx-ingress-controller
        ...
        volumeMounts:
        - mountPath: /etc/nginx/template
          name: nginx-template-volume
          readOnly: true
      ....
      volumes:
      - name: nginx-template-volume
        configMap:
          name: xwz-nginx-template
          items:
          - key: nginx.tmpl
            path: nginx.tmpl
```
至此问题解决
### 监控篇 monitoring
在现有的CooHua API网关上，已经有比较详细的各种指标监控，由于ingress-controller也暴露了prometheus的metrics，所以也很简单的直接部署了社区版的dashboard，并对其进行了简单的修改，如下图所示：
![](http://pp2egchi0.bkt.clouddn.com/FogzrP8HpPTmXz-CR4Gy6WrBQ5D4)
![](http://pp2egchi0.bkt.clouddn.com/FkkB6_9BRIDsV9vWhuqR4rqReMLY)
监控部署的参考文档奉上：[请点击我](https://kubernetes.github.io/ingress-nginx/user-guide/monitoring/)
自定义的dashboard上传到我个人的github上了：[请点击我](https://github.com/zrbcool/blog-public/blob/master/k8s%20ingress%20resources/customized%20ingress%20dashboard.json)
### troubleshooting
#### 抓包
#### dump nginx.conf文件
### refs
http://bogdan-albei.blogspot.com/2017/09/kernel-tuning-in-kubernetes.html
https://danielfm.me/posts/painless-nginx-ingress.html
https://www.asykim.com/blog/deep-dive-into-kubernetes-external-traffic-policies
