```bash
# tree /etc/supervisor/
/etc/supervisor/
├── conf.d #每一个都是具体应用的配置文件
│   ├── alertmanager.conf
│   ├── dingtalk.conf
│   └── prometheus.conf
├── data
│   ├── nflog
│   └── silences
└── supervisord.conf #主配置文件

```
打开一个具体的看一下
```bash
# cat conf.d/prometheus.conf 
[program:prometheus] #起个名字
# 启动命令
command = /app/3rd/prometheus/prometheus --storage.tsdb.retention=180d --config.file=/app/3rd/prometheus/conf/prometheus.yml --web.enable-lifecycle --storage.tsdb.path=/data/coohua/prometheus/data/
user = coohua #执行用户
priority = 1 #优先级
numprocs = 1
autostart = true #自动启动                  
autorestart = true #自动重启      
startretries = 100 #启动重试次数   
stopsignal = KILL #停止信号
stopwaitsecs = 2  #停止等待时间
redirect_stderr = true #重定向标准错误输出
stdout_logfile_maxbytes = 50MB #标准输出每个文件大小
stdout_logfile_backups  = 10 #保存多少个标准输出日志文件
stdout_logfile = /app/3rd/prometheus/supervisor.log #标准输出的日志文件

```
如何启动
```bash
/bin/supervisord -c supervisord.conf
```
查看状态
```bash
# supervisorctl status
alertmanager                     RUNNING   pid 2517, uptime 38 days, 5:06:43
dingtalk                         RUNNING   pid 2518, uptime 38 days, 5:06:43
prometheus                       RUNNING   pid 4903, uptime 17 days, 2:03:12
```