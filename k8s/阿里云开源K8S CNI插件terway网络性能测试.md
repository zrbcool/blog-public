>作者：张荣滨，酷划在线后端架构师，关注微服务治理，容器化技术，Service Mesh等技术领域
微信公众号：**[还没想好名字行吗](#jump_10)**
欢迎关注我的公众号，文章不定期更新;

### terway网络性能测试
酷划在线成立于2014年，是国内激励广告行业的领军者。酷划致力于打造一个用户、广告主、平台三方共赢的激励广告生态体系，旗下产品“酷划锁屏”“淘新闻”分别为锁屏、资讯行业的领跑者。

伴随着公司服务端架构向微服务演进的过程中，服务增多，运维成本提高，资源利用率低，等问题日益凸显，目前公司服务器规模超过700+台ECS，服务数量1000+，随着容器化技术的成熟，计划在近期大规模将生产环境迁移到阿里云容器服务平台上，但由于VxLan等主机转发模式的Overlay网络均有一定的性能损耗，所以我们将目光瞄准阿里云容器服务平台开源的terway网络插件，期望使用能够动态绑定弹性网卡的容器服务达到ECS的网络性能，进而对terway网络性能进行详细的评估。

### 测试说明
本测试基于阿里云容器服务Kubernetes版（1.12.6-aliyun.1），Kubernetes集群使用阿里云控制台创建，测试分两部分：
- 同可用区网络性能测试
- 跨可用区网络性能测试

本测试的所有网络流量均为跨节点通信（容器分布在不同的宿主机节点上）
本测试的所有测试均穿插测试超过3组取结果平均值

### 关键指标
- 吞吐量（Gbit/sec）
- PPS（Packet Per Second）
- 延时（ms）
### 测试方法
吞吐量，PPS测试使用iperf3 
版本信息如下：
```
iperf 3.6 (cJSON 1.5.2)
Linux iperf3-terway-57b5fd565-bwc28 3.10.0-957.5.1.el7.x86_64 #1 SMP Fri Feb 1 14:54:57 UTC 2019 x86_64
Optional features available: CPU affinity setting, TCP congestion algorithm setting, sendfile / zerocopy, socket pacing
```
测试机命令：
```
# 启动服务器模式，暴露在端口16000，每1秒输出一次统计数据
iperf3 -s -i 1 -p 16000
```
陪练机命令：
```
# 测试吞吐量
# 客户端模式，默认使用tcp通信，目标机为172.16.13.218，持续时间45，-P参数指定网卡队列数为4（跟测试的机型有关），目标端口16000
iperf3 -c 172.16.13.218 -t 45 -P 4 -p 16000
# 测试PPS
# 客户端模式，使用udp发包，包大小为16字节，持续时间45秒，-A指定CPU亲和性绑定到第0个CPU
iperf3 -u -l 16 -b 100m -t 45 -c 172.16.13.218 -i 1 -p 16000 -A 0
# 测试延迟
# ping目标机30次
ping -c 30 172.16.13.218
```

## 测试结果
### 同可用区网络性能测试
#### 机型说明
**测试机型选用ecs.sn1ne.2xlarge，规格详情如下**
![](http://oss.zrbcool.top/Fh3HhlfHmLAGPTJFpvZjEDzPCBy2)

#### 测试结果

说明：纵轴表达流量流出方向，横轴表达流量流入方向，所以组合情况一共有9种

![](http://oss.zrbcool.top/FqPXS2zNmi4ySOjTcI2Ck2W1B6eg)
名词解释：
- terway-eni：代表动态创建弹性网卡并绑定POD的terway网络模式
- terway：代表默认的terway网络模式	
#### 结果解读
- 各种模式下均可将网卡带宽打满，从吞吐量上看结果无明显区别
- 从流量流入容器角度看数据，流向terway-eni模式在各项指标均接近甚至超过流向宿主机的性能
- 从流量流出容器角度看数据，terway-eni模式性能接近但略低于宿主机流量流出性能，但明显高于terway默认网络

### 跨可用区网络性能测试
测试机型选用ecs.sn1ne.8xlarge，规格详情如下

![](http://oss.zrbcool.top/FhU8GsIVZqhMDAOTHz0vod8dJq6D)

#### 测试结果

说明：纵轴表达流量流出方向，横轴表达流量流入方向，所以组合情况一共有9种

![](http://oss.zrbcool.top/FotUG6Qumrh1U3VNCkE-rLv_B7e_)

名词解释：
- terway-eni：代表动态创建弹性网卡并绑定POD的terway网络模式
- terway：代表默认的terway网络模式			
#### 结果解读
- 由于增加了跨可用区的调用，使影响结果的因素变多
- host to host的吞吐量，并没有达到网卡的理论最大值，但是流入terway-eni的吞吐量基本达到了机型的带宽6 Gbit/sec，需要进一步调查宿主机间吞吐量上不去的原因
- 从容器流出的流量角度看，terway-eni模式整体明显高于terway默认网络模式，但低于宿主机网络性能
- 从流入容器的流量角度看，terway-eni的PPS结果数据优势比较明显，接近甚至超越宿主机网卡性能

### 总体结论
terway的网络性能测试中表现出了与宣传一致的性能，通过与作者的沟通中了解到，由于将弹性网卡直接放入POD的namespace内，虽然网卡驱动的中断依然由宿主机内核完成，但是网络包不会出现在宿主机namespace的网络栈，减少了宿主机的一层cni网桥转发及复杂路由的性能损失，这也是为什么在某些场景下超过宿主机网络栈性能的表现。


### 作者其他文章
[https://github.com/zrbcool/blog-public](https://github.com/zrbcool/blog-public)  
### 微信订阅号
![](http://oss.zrbcool.top/Fv816XFbZB2JQazo5LHBoy2_SGVz)
 
