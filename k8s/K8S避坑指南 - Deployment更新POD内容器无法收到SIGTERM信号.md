## 简述

容器化后，在应用发布时，某个服务重启，导致该服务调用方大量报错，直到服务重启完成。报错的内容是RPC调用失败，我们的RPC这块是有优雅关闭的，也就是说，在进程收到SIGTERM信号后，我们通过JVM的ShutdownHook机制，注册了RPC服务的反注册钩子，在进程收到SIGTERM时应用会主动从注册中心摘除自身防止调用方大量报错。但是为什么容器化后会导致这个问题呢？

### 问题排查

应用正常启动

![](http://oss.zrbcool.top/picgo/dockerize-2019-06-20-sigkill-1.png)查看容器内进程

```java
# yum install psmisc
# pstree -p
bash(1)───java(22)─┬─{java}(23)
                   ├─{java}(24)
                   ├─{java}(25)
                   ├─{java}(26)
                   ├─{java}(27)
                   ├─{java}(28)
                   ├─{java}(29)
                   ├─{java}(30)
                    ...
# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 09:50 ?        00:00:00 /bin/bash run.sh start
root        22     1 15 09:50 ?        00:01:20 /app/3rd/jdk/default/bin/java -Xmx512m -Xms512m ...
root        49     0  0 09:51 pts/0    00:00:00 bash
root       263    49  0 09:59 pts/0    00:00:00 ps -ef
```

在容器内正常kill 22子进程，可见我们应用的shutdown钩子可以正确处理善后工作

![](http://oss.zrbcool.top/picgo/dockerize-2019-06-20-sigkill-2.png)但是，在实际生产中，我们的deploy滚动更新时，通过查看被删除pod的日志，发现pod被terminate的时候，应用进程并未正确处理SIGTERM信号，问题产生。

### 问题分析

根据对Kubernetes机制的调研，如图： [](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods)[](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods)[](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods)[https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods)

![](http://oss.zrbcool.top/picgo/dockerize-2019-06-20-sigkill-3.png)因为我们的容器是通过run.sh脚本启动，这个在前面截图可以看到，java进程是1号run.sh进程的子进程，对应Kubernetes原理，可知22号java进程在POD删除时不一定会收到SIGTERM，所以导致了我们的shutdown hook不生效。

### 问题解决

既然已经定位问题，那么解决问题的方法就有了思路，run.sh执行java进程后，将进程上下文让给java进程，java进程接管，java进程变为容器内的1号进程。 我们参考了这篇文章受到启发 [](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html)[](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html)[](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html)[https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html)在run.sh执行java前面增加exec命令即可

![](http://oss.zrbcool.top/picgo/dockerize-2019-06-20-sigkill-4.png)然后，重新build镜像，发布，查看进程，发现我们的java进程已经是1号进程

![](http://oss.zrbcool.top/picgo/dockerize-2019-06-20-sigkill-6.png)然后重启，再查看重启前POD留下的日志

![](http://oss.zrbcool.top/picgo/dockerize-2019-06-20-sigkill-5.png)问题解决！
