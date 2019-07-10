
load高
```bash
Tasks: 386 total,   1 running, 385 sleeping,   0 stopped,   0 zombie
%Cpu(s): 35.0 us,  8.6 sy,  0.0 ni, 50.1 id,  0.1 wa,  0.0 hi,  6.1 si,  0.0 st
KiB Mem : 13186283+total, 19950844 free, 51481668 used, 60430324 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 79305216 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                   
16686 root      20   0   13.8g   3.7g  14380 S 121.2  3.0 816:59.58 java                                                                                                                                                                      
30301 root      20   0   13.8g   4.3g  14540 S 100.3  3.4   4901:34 java                                                                                                                                                                      
 7029 root      20   0   13.9g   4.2g  14564 S  83.4  3.3   3561:36 java                                                                                                                                                                      
 8144 root      20   0   14.0g   4.3g  14424 S  81.1  3.4   3588:58 java                                                                                                                                                                      
 4853 root      20   0   14.1g   5.0g  14552 S  80.1  3.9   3533:04 java                                                                                                                                                                      
 9928 root      20   0   14.0g   4.2g  14564 S  77.8  3.3   3585:36 java                                                                                                                                                                      
 7605 root      20   0   13.9g   4.8g  14572 S  65.9  3.8 449:26.70 java                                                                                                                                                                      
22415 root      20   0   12.2g   2.7g  14744 S  42.7  2.2   4085:16 java                                                                                                                                                                      
21773 root      20   0   12.2g   2.7g  14744 S  42.4  2.1   4101:37 java                                                                                                                                                                      
22072 root      20   0   12.2g   2.7g  14684 S  42.4  2.1   4080:20 java                                                                                                                                                                      
 8777 root      20   0 3069764 117676  25776 S   8.6  0.1 663:11.21 dockerd
```
```bash
# ps -e -L h o state,cmd  | awk '{if($1=="R"||$1=="D"){print $0}}' | sort | uniq -c
      8 R /app/3rd/jdk/default/bin/java -Xms4096M -Xmx4096M -XX:+UseG1GC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -verbose:gc -Xloggc:/data/coohua/logs/gc.log -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -Dahas.namespace=prod -Dproject.name=ad-strategy -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:ParallelGCThreads=4 -verbose:gc -Xloggc:/data/coohua/logs/ad-strategy-impl-prod-75c86bb9b8-qxgrq/gc.log -Dapp.logging.path=/data/coohua/logs/ad-strategy-impl-prod-75c86bb9b8-qxgrq -Dcsp.sentinel.log.dir=/data/coohua/logs/ad-strategy-impl-prod-75c86bb9b8-qxgrq -Denv=pro -Dapp.id=ad.strategy -jar /app/coohua/apps/ad-strategy-impl-3.2.46-RELEASE.jar
      2 R /app/3rd/jdk/default/bin/java -Xms4096M -Xmx4096M -XX:+UseG1GC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -verbose:gc -Xloggc:/data/coohua/logs/gc.log -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -Dahas.namespace=prod -Dproject.name=ad-user -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:ParallelGCThreads=4 -verbose:gc -Xloggc:/data/coohua/logs/ad-user-impl-prod-64fc8d6bbd-j5fwb/gc.log -Dapp.logging.path=/data/coohua/logs/ad-user-impl-prod-64fc8d6bbd-j5fwb -Dcsp.sentinel.log.dir=/data/coohua/logs/ad-user-impl-prod-64fc8d6bbd-j5fwb -Denv=pro -Dapp.id=ad.user -jar /app/coohua/apps/ad-user-impl-3.2.32-RELEASE.jar
      2 R /app/3rd/jdk/default/bin/java -Xms4096M -Xmx4096M -XX:+UseG1GC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -verbose:gc -Xloggc:/data/coohua/logs/gc.log -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -Dahas.namespace=prod -Dproject.name=ad-web -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:ParallelGCThreads=4 -verbose:gc -Xloggc:/data/coohua/logs/ad-web-impl-prod-d55c476b7-7whwq/gc.log -Dapp.logging.path=/data/coohua/logs/ad-web-impl-prod-d55c476b7-7whwq -Dcsp.sentinel.log.dir=/data/coohua/logs/ad-web-impl-prod-d55c476b7-7whwq -Denv=pro -Dapp.id=ad.web -jar /app/coohua/apps/ad-web-3.2.44-RELEASE.jar
      1 R /app/3rd/jdk/default/bin/java -Xms4096M -Xmx4096M -XX:+UseG1GC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -verbose:gc -Xloggc:/data/coohua/logs/gc.log -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -Dahas.namespace=prod -Dproject.name=ad-web -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:ParallelGCThreads=4 -verbose:gc -Xloggc:/data/coohua/logs/ad-web-impl-prod-d55c476b7-qwgc2/gc.log -Dapp.logging.path=/data/coohua/logs/ad-web-impl-prod-d55c476b7-qwgc2 -Dcsp.sentinel.log.dir=/data/coohua/logs/ad-web-impl-prod-d55c476b7-qwgc2 -Denv=pro -Dapp.id=ad.web -jar /app/coohua/apps/ad-web-3.2.44-RELEASE.jar
      1 R /app/3rd/jdk/default/bin/java -Xmx2048m -Xms2048m -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+DisableExplicitGC -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -XX:+UseConcMarkSweepGC -XX:+HeapDumpBeforeFullGC -XX:+HeapDumpAfterFullGC -XX:HeapDumpPath=/data/coohua/logs/dump -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -server -Dahas.namespace=prod -Dproject.name=tapweb -verbose:gc -Xloggc:/data/coohua/logs/tap-web-canary-5894dd4f68-rfgbn/gc.log -Dapp.logging.path=/data/coohua/logs/tap-web-canary-5894dd4f68-rfgbn -Dcsp.sentinel.log.dir=/data/coohua/logs/tap-web-canary-5894dd4f68-rfgbn -Denv=pro -Dapp.id=tap -jar /app/coohua/apps/tap-api-2.0.41-RELEASE.jar
      1 R /app/3rd/jdk/default/bin/java -Xmx2048m -Xms2048m -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+DisableExplicitGC -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -XX:+UseConcMarkSweepGC -XX:+HeapDumpBeforeFullGC -XX:+HeapDumpAfterFullGC -XX:HeapDumpPath=/data/coohua/logs/dump -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20M -XX:NumberOfGCLogFiles=12 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintHeapAtGC -server -Dahas.namespace=prod -Dproject.name=tapweb -verbose:gc -Xloggc:/data/coohua/logs/tap-web-prod-cb8c9cc74-v65fh/gc.log -Dapp.logging.path=/data/coohua/logs/tap-web-prod-cb8c9cc74-v65fh -Dcsp.sentinel.log.dir=/data/coohua/logs/tap-web-prod-cb8c9cc74-v65fh -Denv=pro -Dapp.id=tap -jar /app/coohua/apps/tap-api-2.0.41-RELEASE.jar
      1 R ps -e -L h o state,cmd

```

strace -C -f -p 16686
```bash
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 59.74  428.359882        5010     85499     15041 futex
 32.01  229.546681       16067     14287           epoll_wait
  3.89   27.922614      306842        91        19 restart_syscall
  1.55   11.083784         450     24622      6385 read
  1.30    9.340411         447     20915           write
  0.52    3.705619      308802        12        11 accept
  0.23    1.626454         474      3434           sendto
  0.22    1.582314         455      3481           poll
  0.21    1.532525         462      3315           recvfrom
  0.07    0.517175         447      1157           lseek
  0.05    0.371966         449       829           fstat
  0.04    0.268677         409       657           fcntl
  0.03    0.211013         427       494           setsockopt
  0.02    0.157217         468       336       167 stat
  0.02    0.143478         443       324       162 shutdown
  0.02    0.138293         412       336           close
  0.01    0.082764         496       167       163 connect
  0.01    0.078409         464       169           socket
  0.01    0.070391         427       165           getsockname
  0.01    0.069359         415       167           open
  0.01    0.062503         383       163           getsockopt
  0.01    0.062025         376       165           dup2
  0.00    0.020986         356        59           mprotect
  0.00    0.015654         447        35           getrusage
  0.00    0.012555         392        32           sched_getaffinity
  0.00    0.012495         255        49           rt_sigprocmask
  0.00    0.009921         472        21         1 epoll_ctl
  0.00    0.009210         271        34           mmap
  0.00    0.006256          68        92           sched_yield
  0.00    0.004284         268        16           set_robust_list
  0.00    0.004133          75        55           rt_sigreturn
  0.00    0.004092         256        16           clone
  0.00    0.002862         179        16           gettid
  0.00    0.001952         325         6           ioctl
  0.00    0.000988         494         2           statfs
  0.00    0.000110         110         1           madvise
  0.00    0.000035          18         2           times
  0.00    0.000000           0         1           munmap
------ ----------- ----------- --------- --------- ----------------
100.00  717.039087                161222     21949 total
```