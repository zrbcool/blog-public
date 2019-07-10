```bash
# top

top - 11:19:44 up 186 days, 20:06,  1 user,  load average: 43.68, 44.13, 40.66
Tasks: 139 total,   1 running, 138 sleeping,   0 stopped,   0 zombie
%Cpu(s): 18.1 us,  0.4 sy,  0.0 ni, 81.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16267428 total,  2395816 free,  7716248 used,  6155364 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  8181052 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                   
 4903 coohua    20   0  0.682t 7.697g 964036 S 141.2 49.6  61951:49 prometheus                                                                                                                                                                
 2517 root      20   0   31712  16768   4652 S   3.3  0.1 284:50.73 alertmanager                                                                                                                                                              
 2512 root      20   0  222268  15280   1324 S   0.3  0.1  21:37.22 supervisord                                                                                                                                                               
 6816 root      20   0       0      0      0 S   0.3  0.0   0:09.83 kworker/1:2                                                                                                                                                               
12802 root      20   0  162020   2348   1600 R   0.3  0.0   0:00.01 top                                                                                                                                                                       
31007 root       0 -20  128484   7668   4860 S   0.3  0.0 231:10.20 AliYunDun                                                                                                                                                                 
    1 root      20   0   43548   3732   2408 S   0.0  0.0   1:56.06 systemd                                                                                                                                                                   
    2 root      20   0       0      0      0 S   0.0  0.0   0:32.37 kthreadd                                                                                                                                                                  
    3 root      20   0       0      0      0 S   0.0  0.0   0:57.60 ksoftirqd/0
```
iostat -d -x -m 1 1000
```bash
# iostat -d -x -m 1 1000
Linux 3.10.0-514.26.2.el7.x86_64 (monitor-storage001) 	07/08/2019 	_x86_64_	(8 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.64    0.06    1.33     0.00     0.01    17.52     0.01    7.71   23.99    6.95   0.86   0.12
vdb               0.01     0.36    4.03    1.88     0.63     0.64   440.88     0.04    7.42   10.12    1.62   1.35   0.80
vdc               0.00     0.00    0.05    0.04     0.00     0.00    42.95     0.00    4.57    2.39    7.53   0.27   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     5.00    0.00    2.00     0.00     0.03    28.00     0.00    1.00    0.00    1.00   1.00   0.20
vdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
vdc               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```
sar -n DEV 1
```bash
# sar -n DEV 1
Linux 3.10.0-514.26.2.el7.x86_64 (monitor-storage001) 	07/08/2019 	_x86_64_	(8 CPU)

11:17:05 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11:17:06 AM      eth0     59.00     20.00     71.47      1.66      0.00      0.00      0.00
11:17:06 AM        lo     72.00     72.00      3.23      3.23      0.00      0.00      0.00

11:17:06 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11:17:07 AM      eth0    192.00     36.00    262.68      3.85      0.00      0.00      0.00
11:17:07 AM        lo     50.00     50.00      2.25      2.25      0.00      0.00      0.00

11:17:07 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11:17:08 AM      eth0     94.00     52.00    100.77      4.78      0.00      0.00      0.00
11:17:08 AM        lo     61.00     61.00      2.75      2.75      0.00      0.00      0.00

11:17:08 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
11:17:09 AM      eth0    115.00     42.00    139.66      3.28      0.00      0.00      0.00
11:17:09 AM        lo     42.00     42.00     16.37     16.37      0.00      0.00      0.00
```
ps -e -L h o state,cmd  | awk '{if($1=="R"||$1=="D"){print $0}}' | sort | uniq -c | sort -k 1nr
```bash
# ps -e -L h o state,cmd  | awk '{if($1=="R"||$1=="D"){print $0}}' | sort | uniq -c | sort -k 1nr
     40 D /app/3rd/node_exporter/node_exporter --collector.meminfo_numa --collector.tcpstat
      2 R /app/3rd/prometheus/prometheus --storage.tsdb.retention=180d --config.file=/app/3rd/prometheus/conf/prometheus.yml --web.enable-lifecycle --storage.tsdb.path=/data/coohua/prometheus/data/
      1 D [172.16.38.238-m]
      1 D -bash
      1 R ps -e -L h o state,cmd
```