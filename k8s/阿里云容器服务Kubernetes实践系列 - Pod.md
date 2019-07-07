>作者：荣滨，酷划在线后端架构师，关注微服务治理，容器化技术，Service Mesh等技术领域

### 背景篇
上篇文章[阿里云容器服务Kubernetes实践系列 - Ingress篇](https://yq.aliyun.com/articles/699445)阐述了我们的流量导入路径，描述了我们如何使用SLB及Ingress完成四层/七层的负载均衡，本篇文章我们来重点看下以下几个方面：  
- POD的组成
- POD的启动入口设计
- POD的初始化过程
- POD的销毁过程
#### POD的组成
我们的业务服务的POD一般包含以下几个部分，Probe、PVC、ConfigMap、Resource等部分组成，接下来我们来一一分解一下
```yaml
spec:
  containers:
  - name: ad-web-impl
    image: registry.cn-qingdao.aliyuncs.com/xxx/xxxx:19-814bbe98
    imagePullPolicy: IfNotPresent #如果本地存在则不会拉取远端镜像
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
    - containerPort: 9145
      name: exporter
      protocol: TCP
    readinessProbe:
      failureThreshold: 10
      httpGet:
        path: /actuator/health
        port: 8080
        scheme: HTTP
      initialDelaySeconds: 30
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      limits:
        cpu: "4"
        memory: 6Gi
      requests:
        cpu: 500m
        memory: 3865470566400m
    envFrom:
    - configMapRef:
        name: ad-web-impl-prod
    volumeMounts:
    - mountPath: /data
      name: tomcat-data
    securityContext:
      privileged: false
      procMount: Default
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
  dnsPolicy: ClusterFirst
  imagePullSecrets:
  - name: coohua
  restartPolicy: Always
  terminationGracePeriodSeconds: 30
  volumes:
  - name: tomcat-data
    persistentVolumeClaim:
      claimName: ad-web-impl-prod
```
上面的yaml片段描述的是我们的POD主体描述部分这部分其实是每个POD的核心，目前我们还没有集成任何Sidecar，所以我们的POD的核心container只有一个，也就是我们的业务主线程
##### containers
- name  
这个没什么好说的，每个container都要取一个名字，如果POD有多个container，那么kubectl logs等命令时会让指定container来操作某个具体的container，多个container之间默认只共享network namespace
- image  
我们使用阿里云的容器镜像服务托管镜像，镜像的名字是这样的格式：registry.cn-qingdao.aliyuncs.com/[namespace]/[imageName]:[buildNumber]-[gitCommitHash]
namespace及imageName用来定位是哪个服务，buildNumber是jenkins构建的jobNumber，gitCommitHash是代码提交点的hash值，可以用于追溯
- port  
8080用于暴露http接口，9145是prometheus监控数据的endpoint，由于内部服务之间的调用走TCP的RPC协议，POD之间端口默认都是通的，所以无需特别配置
- resource  
资源这块后面会有文章专门阐述，这里简单说就是通过对服务规律的观测人工调整合理的资源分配比例
- readinessProbe  
这块后面讲滚动升级的部分会详细描述，简单说就是应用内部暴露8080端口的一个http服务用于服务可用性检查，当接口结果返回200时再接入http流量
- envFrom
这个比较重要，这里重点说一下，这个特性的作用是将configMap中的key，value作为环境变量注入到POD当中，这对于我们标准化镜像十分有用，我们将一些无法放入配置管理中心（我们使用Apollo）的参数通过环境变量的方式传递给POD，例如Apollo的appID，JVM启动参数，熔断限流库Sentinel的服务器地址等。这里我把ConfigMap的内容也贴一下  
- volumeMounts  
保存日志的存储卷挂载信息
```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: ad-web-impl-prod
  namespace: coohua
  labels:
    app: ad-web-impl-prod
    wayne-app: ad-web-impl
    wayne-ns: coohua
data:
  denv: pro
  appid: ad.web
  DEBUG: ' '
  JAVA_OPTIONS: >-
    -Xms4096M -Xmx4096M -XX:+UseG1GC -XX:+PrintGCApplicationStoppedTime
    -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics
    -XX:PrintSafepointStatisticsCount=1 -verbose:gc
    -Xloggc:/data/coohua/logs/gc.log -XX:+UseGCLogFileRotation
    -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails
    -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC
    -Dahas.namespace=prod -Dproject.name=ad-web -XX:+UnlockExperimentalVMOptions
    -XX:+UseCGroupMemoryLimitForHeap -XX:ParallelGCThreads=4
  sentinel_server: ' '
```





```yaml
spec:
  containers:
  - envFrom:
    - configMapRef:
        name: ad-web-impl-prod
    image: registry.cn-qingdao.aliyuncs.com/xxx/xxxx:19-814bbe98
    imagePullPolicy: IfNotPresent
    name: ad-web-impl
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
    - containerPort: 9145
      name: exporter
      protocol: TCP
    readinessProbe:
      failureThreshold: 10
      httpGet:
        path: /actuator/health
        port: 8080
        scheme: HTTP
      initialDelaySeconds: 30
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      limits:
        cpu: "4"
        memory: 6Gi
      requests:
        cpu: 500m
        memory: 3865470566400m
    securityContext:
      privileged: false
      procMount: Default
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /data
      name: tomcat-data
  dnsPolicy: ClusterFirst
  imagePullSecrets:
  - name: coohua
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  terminationGracePeriodSeconds: 30
  volumes:
  - name: tomcat-data
    persistentVolumeClaim:
      claimName: ad-web-impl-prod
```