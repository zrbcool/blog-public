>作者：荣滨，酷划在线后端架构师，关注微服务治理，容器化技术，Service Mesh等技术领域

### 背景篇
#### 现状
酷划在线成立于2014年，是国内激励广告行业的领军者。酷划致力于打造一个用户、广告主、平台三方共赢的激励广告生态体系，旗下产品“酷划锁屏”“淘新闻”分别为锁屏、资讯行业的领跑者。

前面一篇文章主要是落地容器化之前对基础网络组件的调研及性能测试，感兴趣的同学请参考：[阿里云开源K8S CNI插件terway网络性能测试](https://yq.aliyun.com/articles/696639?spm=a2c4e.11153940.bloghomeflow.62.2b9f291als77PA&do=login&accounttraceid=ea8e5d9d-befd-49b7-822a-3a8f413bb2e7)

目前公司的后端架构基本上是微服务的架构模式，如下图，所有的入站流量均通过API网关进入到后端服务，API网关起到一个“保护神”的作用，监控着所有进入的请求，并具备防刷，监控接口性能，报警等重要功能，流量通过网关后服务间的调用均为RPC调用。
![](http://pp2egchi0.bkt.clouddn.com/FtqXlQ2CZ6kHd1Rfbf6JRzYxrTIJ)
#### 过渡期
在经历完微服务改造后，虽然微服务架构给我们带来了不少红利，但是也不免引入一些问题： 
- 服务拆分导致机器数增多 
- 为了降低资源浪费，多个服务部署到一台主机造成的端口管理成本
- 不一致的交付环境，造成的排查问题成本

为了解决上述问题，经过调查，我们将目光锁定容器服务Kubernetes，以解决我们的问题，本篇文章主要关注nginx-ingress-controller（后面统一简称NGINX IC）部分，所以下面的架构主要突出API网关及IC。
下图是我们的过渡期方案：
通过引入一个内网的SLB来解决IC做为我们API网关upstream时，服务发现的问题。并且，可以通过逐步切换接口到SLB上达到渐进式迁移的效果，粒度可以做到接口+百分比级别。
过渡期间架构图如下所示：

![](http://pp2egchi0.bkt.clouddn.com/FtFETZ7Ngjn7iomBzcTxUN9dk8qc)
#### 终态
全部迁移完成后，所有的机器回收，如下图所示：
![](http://pp2egchi0.bkt.clouddn.com/FkN4eqHNFlqruc8Z7oG_SfKllhgs)
其实两层SLB是会有浪费的，但由于我们的API网关目前承担着重要的作用，必须找到替代方案才能去掉。
我们也尝试调研了用定制IC，或者Service Mesh架构的Istio方案来解决，由于当时Istio 1.1还没发布，并且大规模使用的案例太少，所以我们保守的选择了目前的这种折中方案，后续要切换到Istio上也不是很麻烦。
#### 按业务线分组IC
![](http://pp2egchi0.bkt.clouddn.com/FnmAS0JoxHAx9PT8PcJghkmUPUUY)
默认Kubernetes所有的流量都由一组默认的IC来承担，这个方案我们从一开始就是否定的，所以通过部署多组IC来达到一定的隔离是必要的。最终会是上面图的那个形态。
### 实践篇
#### 如何启用多IC
查看IC的启动命令参数，可以找到：
![](http://pp2egchi0.bkt.clouddn.com/Fl4jrnaE1UOSKQSp-yZ7Oa2jynun)
其意思是只关注携带annotation为"kubernetes.io/ingress.class"，并且与IC启动参数参数--ingress-class的值相同的Ingress定义，不符合的会被忽略，这样就做到了多组IC，Ingress定义之间的隔离。
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
接下来我们来看下IC在Kubernetes内部署的资源结构，如图所示：
![](http://pp2egchi0.bkt.clouddn.com/FhnUMZTT4ll37v4qUjIZSssEMwjT)
可以看到一组IC其实由以下几部分组成：
- ServiceAccount，ClusterRole，ClusterRoleBinding：权限RBAC定义
- Deployment：控制controller的部署，并依赖ServiceAccount，Configmap，Service
- ConfigMap：三个configmap都是保存自定义的controller配置用的
- Service：这里使用type为LoadBalancer的svc主要是利用阿里云基础服务实现的自动绑定到SLB实例的功能

我们的场景是不需要特殊的权限配置的，所以我们就简单的把红框里面的几个资源复制一份修改下其中的几个配置（例如--ingress-class=xwz），然后直接引用默认IC的ServiceAccount，就完成了一组全新IC的部署，并且与Kubernetes自带的IC是互相隔离的。
这里我把我写好的配置放到我的github上，读者可以参考：[ingress resources](https://github.com/zrbcool/blog-public/tree/master/k8s%20ingress%20resources) 

创建好新IC后，如果发现IC有如下错误
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
#### 扩容的IC实例如何自动添加到SLB的后端服务列表中
##### 方式一 externalTrafficPolicy=Cluster
Service的spec.externalTrafficPolicy当为Cluster的时候：
集群当中的每台主机都可以充当三层路由器，起到负载均衡及转发的作用，但是由于其对请求包进行了SNAT操作，如图所示：
![](http://pp2egchi0.bkt.clouddn.com/Fn5Lpn7EUvC7xXOUCkNIHCHpDbNB)
这样导致IC的POD内获取到的client-ip会是转发包过来的那个worker节点的IP，所以在这个模式下我们无法获取客户端的真实IP，如果我们不关心客户端真实IP的情况，可以使用这种方式，然后将所有的worker节点IP加入到SLB的后端服务列表当中即可。
##### 方式二 externalTrafficPolicy=Local
Service的spec.externalTrafficPolicy当为Local的时候：
节点只会把请求转给节点内的IC的POD，由于不经过SNAT操作，IC可以获取到客户端的真实IP，如果节点没有POD，就会报错。这样我们就需要手工维护IC POD，节点，与SLB后端服务之间的关系。那么有没有一种方式可以自动管理维护这个关系呢？其实是有的，阿里云容器服务为我们做好了这一切，只要在type为LoadBalancer的Service上增加如下几个annotation，它就可以为我们将启动了POD的worker节点的端口及IP自动添加到SLB后端服务当中，扩容缩容自动更新，如下所示（注意我们使用的是内网SLB，type为intranet，大家根据实际情况修改）：
```
metadata:
  annotations:
    service.beta.kubernetes.io/alicloud-loadbalancer-address-type: intranet
    service.beta.kubernetes.io/alicloud-loadbalancer-force-override-listeners: "true"
    service.beta.kubernetes.io/alicloud-loadbalancer-id: lb-2zec8x×××××××××965vt
```
##### 方式一（Cluster） vs 方式二（Local）
 对比 | Cluster | Local
----|------|----
优点 | 简单，K8S的默认方式  | 减少网络转发，性能好，能获取客户端真实IP
缺点 | SNAT地址伪装网络上增加一跳，性能下降，无法获取客户端真实IP  | 需要对节点的端口是否打开做检查，需要自定义服务发现（阿里云这块已经做了与SLB的集成）
### 绕坑篇
#### nginx worker进程数的问题
我们知道nginx的默认配置worker_processes为auto的时候，会根据当前主机cpu信息自动计算，但是nginx并不是一个cgroups aware的应用，所以其会盲目“自大”的认为有“好多”cpu可以用，这里我们就需要对其进行指定，可以在configmap中设置参数：
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
小流量灰度期间业务同学反馈说第三方的反作弊发现我们调用他们的接口异常，一轮分析下来发现，原来是发给第三方请求中携带的客户端IP被写为了我们API网关的主机IP 
还记得前面的架构图吗？
![](http://pp2egchi0.bkt.clouddn.com/FkN4eqHNFlqruc8Z7oG_SfKllhgs)
出现这个问题的原因就是我们的IC被放到了OpenResty后面，查看到IC的template中对X-REAL-IP设置的代码部分如下：
![](http://pp2egchi0.bkt.clouddn.com/Fk2yvF82fpL6_KP3fkOErj5oZJ5X)
这个the_real_ip又是哪来的呢？
![](http://pp2egchi0.bkt.clouddn.com/Fqle-PiPDR9AZNmgpUDwWB-qZ2oI) 
可以看到默认情况下，是从remote_addr这个变量获取，在我们的场景下remote_addr就会是API网关的主机IP，所以我们需要修改其为API网关为我们设置好的名为X_REAL_IP的header。我们的做法是将template文件导出修改并以congfigmap的方式挂载进镜像，覆盖原来的配置，配置如下：
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
### Troubleshooting篇
#### dump nginx.conf文件
请参考这篇文章
https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#dumping-nginx-configuration-from-a-running-process
### Refs
https://yq.aliyun.com/articles/692732?spm=a2c4e.11163080.searchblog.53.764b2ec1hJYa02
https://yq.aliyun.com/articles/645856?spm=a2c4e.11163080.searchblog.97.764b2ec1hJYa02
http://bogdan-albei.blogspot.com/2017/09/kernel-tuning-in-kubernetes.html
https://danielfm.me/posts/painless-nginx-ingress.html
https://www.asykim.com/blog/deep-dive-into-kubernetes-external-traffic-policies
