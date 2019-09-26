## 开源项目推荐
[Pepper Metrics](https://github.com/zrbcool/pepper-metrics)是我与同事开发的一个开源工具(https://github.com/zrbcool/pepper-metrics)，其通过收集jedis/mybatis/httpservlet/dubbo/motan的运行性能统计，并暴露成prometheus等主流时序数据库兼容数据，通过grafana展示趋势。其插件化的架构也非常方便使用者扩展并集成其他开源组件。  
请大家给个star，同时欢迎大家成为开发者提交PR一起完善项目。

## 前言
前面的文章，我们讲述了如何通过perf的方式对java程序进行性能剖析，并生成FlameGraph火焰图，但是实际生产中，很多企业会将java部署在docker容器当中，这时对docker内运行的java进程进行剖析便成为一件很麻烦的事情。

## 执行步骤
安装相关依赖
```bash
yum install -y git cmake gcc-c++ gcc perf
```
下载项目
```bash
git clone https://github.com/zrbcool/docker-flame-graphs.git
```
指定JAVA_HOME环境变量
```bash
export JAVA_HOME=/root/jdk1.8.0_181
export PATH=$JAVA_HOME/bin:$PATH
```
编译项目
```bash
cd docker-flame-graphs/
cmake . && make
```

找到你要分析的docker进程
```bash
docker ps | grep xxx
```  
找到这个容器的进程Pid
```bash
docker inspect --format '{{.State.Pid}}' [CONTAINER_ID]
```
修改脚本当中的JAVA_HOME保证与容器内部的JAVA_HOME一致  
```bash
vi bin/create-java-perf-map.sh
export JAVA_HOME=/app/3rd/jdk/default
```
去掉脚本当中被注释的命令：  
```bash
vi bin/docker-perf-top
#删掉下面代码前面的注释
sudo perf top -p $host_pid
```
在docker-flame-graphs目录下，执行：  
```bash
docker cp $(pwd) [CONTAINER_ID]:/docker-flame-graphs
```
然后需要确认你的JVM参数增加了-XX:+PreserveFramePointer，如果没有，需要增加并重启服务  
现在所有的准备工作已经完成，让你的JVM进程运行一段时间完成JIT的预热  
然后我们开始分析性能：  
```bash
cd bin
./docker-perf-top [CONTAINER_ID] [JAVA_ID]
./docker-perf-java-flames [CONTAINER_ID] [JAVA_ID]
```
### docker-perf-top效果
![](http://oss.zrbcool.top/picgo/docker-java-perf-top.png)
### docker-perf-java-flames
svg图像可以下钻等操作，请打开链接查看[点我](http://oss.zrbcool.top/picgo/docker-java-perf-flamegraph.svg)  
![](http://oss.zrbcool.top/picgo/docker-java-perf-flamegraph.svg)


### 参考
```bash
https://github.com/jvm-profiling-tools/perf-map-agent/issues/50  
https://blog.alicegoldfuss.com/making-flamegraphs-with-containerized-java/  
https://github.com/mboussaa/docker-flame-graphs  
http://www.batey.info/docker-jvm-flamegraphs.html  
https://github.com/chbatey/perf-map-agent  
https://blog.alicegoldfuss.com/making-flamegraphs-with-containerized-java/  
https://github.com/jvm-profiling-tools/perf-map-agent  
https://medium.com/netflix-techblog/java-in-flames-e763b3d32166  
```
### 作者其他文章
[https://github.com/zrbcool/blog-public](https://github.com/zrbcool/blog-public)  
### 微信订阅号
![](http://oss.zrbcool.top/Fv816XFbZB2JQazo5LHBoy2_SGVz)