### 安装及使用

安装前提软件

#### centos

```java
yum install perf -y
yum install gcc -y
yum install gcc-c++
yum install cmake -y
```

#### ubuntu

```java
apt install linux-tools-generic
apt install linux-tools-common
```

#### FlameGraph

```java
# 参考 http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#Java
export JAVA_HOME=/root/jdk1.8.0_181
export PATH=$JAVA_HOME/bin:$PATH
git clone --depth=1 https://github.com/jrudolph/perf-map-agent
cd perf-map-agent
cmake .
make
bin/create-links-in /usr/bin

git clone https://github.com/brendangregg/FlameGraph.git
export FLAMEGRAPH_DIR=/root/git/FlameGraph
# jvm启动参数需要增加
-XX:+PreserveFramePointer

perf-java-flames 21322
```

### 结果展示

svg格式的图片可以下钻，点击查看: [](http://oss.zrbcool.top/picgo/flamegraph-21322.svg)[http://oss.zrbcool.top/picgo/flamegraph-21322.svg](http://oss.zrbcool.top/picgo/flamegraph-21322.svg)

![](http://oss.zrbcool.top/picgo/flamegraph-21322.svg)

### 容器内如何分析？
请参考笔者的另一篇文章[Docker中使用FlameGraph分析JVM应用性能](https://juejin.im/post/5d3300cf51882539af1922be)
### 更多参考
```bash
https://www.meiwen.com.cn/subject/xafqkqtx.html
https://bugs.openjdk.java.net/browse/JDK-8068945
https://bugs.openjdk.java.net/browse/JDK-6515172
http://mail.openjdk.java.net/pipermail/hotspot-compiler-dev/2014-December/016477.html
https://medium.com/netflix-techblog/java-in-flames-e763b3d32166
https://blog.codecentric.de/en/2017/09/jvm-fire-using-flame-graphs-analyse-performance/
https://github.com/jvm-profiling-tools/honest-profiler/wiki/How-to-Run
https://gperftools.github.io/gperftools/pprof-vsnprintf-big.gif
https://github.com/jvm-profiling-tools/perf-map-agent
http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#Java
http://www.brendangregg.com/blog/2014-06-12/java-flame-graphs.html
https://github.com/brendangregg/FlameGraph
```
