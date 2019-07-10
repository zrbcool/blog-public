### 测试随机写IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```
测试结果如下：
```bash
Rand_Write_Testing: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
fio-3.1
Starting 1 process
Rand_Write_Testing: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=72.8MiB/s][r=0,w=18.6k IOPS][eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=11645: Mon Jul  8 10:56:08 2019
  write: IOPS=22.0k, BW=89.7MiB/s (94.1MB/s)(1024MiB/11412msec)
    slat (usec): min=4, max=21592, avg=19.71, stdev=138.27
    clat (usec): min=381, max=49139, avg=5548.08, stdev=6114.22
     lat (usec): min=392, max=49154, avg=5568.21, stdev=6116.49
    clat percentiles (usec):
     |  1.00th=[ 1090],  5.00th=[ 1614], 10.00th=[ 1958], 20.00th=[ 2442],
     | 30.00th=[ 2835], 40.00th=[ 3261], 50.00th=[ 3720], 60.00th=[ 4228],
     | 70.00th=[ 4948], 80.00th=[ 5997], 90.00th=[10683], 95.00th=[18220],
     | 99.00th=[33817], 99.50th=[35390], 99.90th=[40633], 99.95th=[44303],
     | 99.99th=[47449]
   bw (  KiB/s): min=33976, max=104872, per=99.93%, avg=91820.45, stdev=20494.88, samples=22
   iops        : min= 8494, max=26218, avg=22955.05, stdev=5123.75, samples=22
  lat (usec)   : 500=0.03%, 750=0.22%, 1000=0.48%
  lat (msec)   : 2=9.93%, 4=44.90%, 10=33.99%, 20=5.87%, 50=4.58%
  cpu          : usr=6.95%, sys=38.42%, ctx=73206, majf=0, minf=26
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwt: total=0,262144,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
  WRITE: bw=89.7MiB/s (94.1MB/s), 89.7MiB/s-89.7MiB/s (94.1MB/s-94.1MB/s), io=1024MiB (1074MB), run=11412-11412msec

Disk stats (read/write):
  vdc: ios=0/262670, merge=0/1054, ticks=0/1227246, in_queue=1228423, util=92.27%
```
### 测试随机读IOPS
```bash
fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```
测试结果如下：
```bash
Rand_Read_Testing: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [r(1)][100.0%][r=99.1MiB/s,w=0KiB/s][r=25.4k,w=0 IOPS][eta 00m:00s]
Rand_Read_Testing: (groupid=0, jobs=1): err= 0: pid=11813: Mon Jul  8 10:57:25 2019
   read: IOPS=23.9k, BW=93.2MiB/s (97.7MB/s)(1024MiB/10989msec)
    slat (nsec): min=1973, max=62799k, avg=8617.32, stdev=146894.44
    clat (usec): min=341, max=67750, avg=5353.76, stdev=11460.67
     lat (usec): min=345, max=67753, avg=5362.78, stdev=11460.89
    clat percentiles (usec):
     |  1.00th=[  807],  5.00th=[ 1106], 10.00th=[ 1287], 20.00th=[ 1549],
     | 30.00th=[ 1778], 40.00th=[ 1991], 50.00th=[ 2212], 60.00th=[ 2474],
     | 70.00th=[ 2835], 80.00th=[ 3720], 90.00th=[ 9634], 95.00th=[12649],
     | 99.00th=[60556], 99.50th=[61604], 99.90th=[63177], 99.95th=[64750],
     | 99.99th=[66847]
   bw (  KiB/s): min=56605, max=110176, per=99.69%, avg=95128.81, stdev=15293.25, samples=21
   iops        : min=14151, max=27542, avg=23782.05, stdev=3823.36, samples=21
  lat (usec)   : 500=0.04%, 750=0.63%, 1000=2.38%
  lat (msec)   : 2=37.14%, 4=41.51%, 10=9.57%, 20=4.20%, 50=0.63%
  lat (msec)   : 100=3.91%
  cpu          : usr=5.72%, sys=16.13%, ctx=19871, majf=0, minf=158
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwt: total=262144,0,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
   READ: bw=93.2MiB/s (97.7MB/s), 93.2MiB/s-93.2MiB/s (97.7MB/s-97.7MB/s), io=1024MiB (1074MB), run=10989-10989msec

Disk stats (read/write):
  vdc: ios=260439/571, merge=0/6, ticks=1212852/5445, in_queue=1219518, util=91.04%
```
### 测试顺序写吞吐量
```bash
fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```
测试结果如下
```bash
Write_PPS_Testing: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=64
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [W(1)][-.-%][r=0KiB/s,w=288MiB/s][r=0,w=288 IOPS][eta 00m:00s]
Write_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=11876: Mon Jul  8 10:58:36 2019
  write: IOPS=290, BW=291MiB/s (305MB/s)(1024MiB/3521msec)
    slat (usec): min=82, max=135248, avg=3297.91, stdev=13465.05
    clat (msec): min=48, max=406, avg=215.23, stdev=58.42
     lat (msec): min=48, max=406, avg=218.53, stdev=58.83
    clat percentiles (msec):
     |  1.00th=[   52],  5.00th=[  125], 10.00th=[  142], 20.00th=[  176],
     | 30.00th=[  190], 40.00th=[  203], 50.00th=[  209], 60.00th=[  222],
     | 70.00th=[  241], 80.00th=[  262], 90.00th=[  292], 95.00th=[  326],
     | 99.00th=[  355], 99.50th=[  368], 99.90th=[  405], 99.95th=[  405],
     | 99.99th=[  405]
   bw (  KiB/s): min=204800, max=348160, per=94.41%, avg=281161.14, stdev=53060.12, samples=7
   iops        : min=  200, max=  340, avg=274.57, stdev=51.82, samples=7
  lat (msec)   : 50=0.68%, 100=1.37%, 250=73.93%, 500=24.02%
  cpu          : usr=2.87%, sys=3.89%, ctx=137, majf=0, minf=27
  IO depths    : 1=0.1%, 2=0.2%, 4=0.4%, 8=0.8%, 16=1.6%, 32=3.1%, >=64=93.8%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.9%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwt: total=0,1024,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
  WRITE: bw=291MiB/s (305MB/s), 291MiB/s-291MiB/s (305MB/s-305MB/s), io=1024MiB (1074MB), run=3521-3521msec

Disk stats (read/write):
  vdc: ios=0/2985, merge=0/0, ticks=0/411581, in_queue=422492, util=96.08%
```
### 测试顺序读吞吐量
```bash
fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```
测试结果如下
```bash
Read_PPS_Testing: (g=0): rw=read, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=64
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [R(1)][100.0%][r=290MiB/s,w=0KiB/s][r=289,w=0 IOPS][eta 00m:00s]
Read_PPS_Testing: (groupid=0, jobs=1): err= 0: pid=11953: Mon Jul  8 11:00:03 2019
   read: IOPS=292, BW=292MiB/s (306MB/s)(1024MiB/3505msec)
    slat (usec): min=44, max=89165, avg=3248.11, stdev=11962.95
    clat (msec): min=27, max=380, avg=214.65, stdev=45.36
     lat (msec): min=28, max=380, avg=217.90, stdev=46.40
    clat percentiles (msec):
     |  1.00th=[  100],  5.00th=[  131], 10.00th=[  184], 20.00th=[  197],
     | 30.00th=[  203], 40.00th=[  205], 50.00th=[  207], 60.00th=[  211],
     | 70.00th=[  215], 80.00th=[  253], 90.00th=[  279], 95.00th=[  292],
     | 99.00th=[  372], 99.50th=[  376], 99.90th=[  380], 99.95th=[  380],
     | 99.99th=[  380]
   bw (  KiB/s): min=222786, max=301896, per=93.46%, avg=279584.86, stdev=29293.68, samples=7
   iops        : min=  217, max=  294, avg=272.71, stdev=28.61, samples=7
  lat (msec)   : 50=0.88%, 100=0.59%, 250=77.93%, 500=20.61%
  cpu          : usr=0.11%, sys=4.02%, ctx=238, majf=0, minf=2618
  IO depths    : 1=0.1%, 2=0.2%, 4=0.4%, 8=0.8%, 16=1.6%, 32=3.1%, >=64=93.8%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.9%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwt: total=1024,0,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=292MiB/s (306MB/s), 292MiB/s-292MiB/s (306MB/s-306MB/s), io=1024MiB (1074MB), run=3505-3505msec

Disk stats (read/write):
  vdc: ios=2965/90, merge=0/0, ticks=399765/7315, in_queue=418443, util=97.68%
```
### 测试随机写时延
```bash
fio -direct=1 -iodepth=1 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Write_Latency_Testing
```
测试结果如下
```bash
Rand_Write_Latency_Testing: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=1
fio-3.1
Starting 1 process
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=7811KiB/s][r=0,w=1952 IOPS][eta 00m:00s]
Rand_Write_Latency_Testing: (groupid=0, jobs=1): err= 0: pid=12099: Mon Jul  8 11:06:19 2019
  write: IOPS=1631, BW=6527KiB/s (6684kB/s)(1024MiB/160642msec)
    slat (usec): min=4, max=3834, avg= 9.70, stdev= 8.92
    clat (usec): min=321, max=62045, avg=600.17, stdev=956.17
     lat (usec): min=329, max=62052, avg=610.30, stdev=958.75
    clat percentiles (usec):
     |  1.00th=[  363],  5.00th=[  375], 10.00th=[  383], 20.00th=[  396],
     | 30.00th=[  408], 40.00th=[  416], 50.00th=[  429], 60.00th=[  441],
     | 70.00th=[  461], 80.00th=[  494], 90.00th=[  709], 95.00th=[ 1303],
     | 99.00th=[ 3458], 99.50th=[ 8979], 99.90th=[10945], 99.95th=[11863],
     | 99.99th=[19006]
   bw (  KiB/s): min=  384, max= 8368, per=99.95%, avg=6523.53, stdev=1991.11, samples=321
   iops        : min=   96, max= 2092, avg=1630.84, stdev=497.86, samples=321
  lat (usec)   : 500=81.09%, 750=9.51%, 1000=2.81%
  lat (msec)   : 2=4.17%, 4=1.56%, 10=0.41%, 20=0.45%, 50=0.01%
  lat (msec)   : 100=0.01%
  cpu          : usr=0.73%, sys=2.69%, ctx=262171, majf=0, minf=27
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwt: total=0,262144,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=6527KiB/s (6684kB/s), 6527KiB/s-6527KiB/s (6684kB/s-6684kB/s), io=1024MiB (1074MB), run=160642-160642msec

Disk stats (read/write):
  vdc: ios=0/266307, merge=0/74, ticks=0/172899, in_queue=172866, util=87.68%
```
### 测试随机读时延
```bash
fio -direct=1 -iodepth=1 -rwfio -direct=1 -iodepth=1 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Read_Latency_Testingrandwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -group_reporting -filename=iotest -name=Rand_Write_Latency_Testing
```
测试结果如下
```bash
Rand_Read_Latency_Testingrandwrite: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=1
Rand_Write_Latency_Testing: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=1
fio-3.1
Starting 2 processes
3;fio-3.1;Rand_Read_Latency_Testingrandwrite;0;0;2097152;14790;3697;141787;2;108;8.432369;3.213270;311;35648;525.634168;809.391753;1.000000%=354;5.000000%=366;10.000000%=378;20.000000%=395;30.000000%=407;40.000000%=419;50.000000%=428;60.000000%=440;70.000000%=452;80.000000%=473;90.000000%=518;95.000000%=643;99.000000%=2211;99.500000%=8224;99.900000%=10944;99.950000%=11075;99.990000%=16449;0%=0;0%=0;0%=0;328;35667;534.512884;810.705574;383;8944;50.317445%;7441.950178;2275.304448;0;0;0;0;0;0;0.000000;0.000000;0;0;0.000000;0.000000;1.000000%=0;5.000000%=0;10.000000%=0;20.000000%=0;30.000000%=0;40.000000%=0;50.000000%=0;60.000000%=0;70.000000%=0;80.000000%=0;90.000000%=0;95.000000%=0;99.000000%=0;99.500000%=0;99.900000%=0;99.950000%=0;99.990000%=0;0%=0;0%=0;0%=0;0;0;0.000000;0.000000;0;0;0.000000%;0.000000;0.000000;0.998455%;2.910505%;524312;0;64;100.0%;0.0%;0.0%;0.0%;0.0%;0.0%;0.0%;0.00%;0.00%;0.00%;0.00%;0.00%;0.00%;0.00%;87.09%;9.09%;1.48%;1.24%;0.36%;0.36%;0.38%;0.01%;0.00%;0.00%;0.00%;0.00%;0.00%;0.00%;0.00%;vdc;523946;3;0;1;243086;1;243008;89.18%

Rand_Read_Latency_Testingrandwrite: (groupid=0, jobs=2): err= 0: pid=12354: Mon Jul  8 11:11:56 2019
   read: IOPS=3697, BW=14.4MiB/s (15.1MB/s)(2048MiB/141787msec)
    slat (usec): min=2, max=108, avg= 8.43, stdev= 3.21
    clat (usec): min=311, max=35648, avg=525.63, stdev=809.39
     lat (usec): min=328, max=35667, avg=534.51, stdev=810.71
    clat percentiles (usec):
     |  1.00th=[  355],  5.00th=[  367], 10.00th=[  379], 20.00th=[  396],
     | 30.00th=[  408], 40.00th=[  420], 50.00th=[  429], 60.00th=[  441],
     | 70.00th=[  453], 80.00th=[  474], 90.00th=[  519], 95.00th=[  644],
     | 99.00th=[ 2212], 99.50th=[ 8225], 99.90th=[10945], 99.95th=[11076],
     | 99.99th=[16450]
   bw (  KiB/s): min=  383, max= 8944, per=50.32%, avg=7441.95, stdev=2275.30, samples=562
   iops        : min=   95, max= 2236, avg=1860.45, stdev=568.90, samples=562
  lat (usec)   : 500=87.09%, 750=9.09%, 1000=1.48%
  lat (msec)   : 2=1.24%, 4=0.36%, 10=0.36%, 20=0.38%, 50=0.01%
  cpu          : usr=1.00%, sys=2.91%, ctx=524312, majf=0, minf=64
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwt: total=524288,0,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: bw=14.4MiB/s (15.1MB/s), 14.4MiB/s-14.4MiB/s (15.1MB/s-15.1MB/s), io=2048MiB (2147MB), run=141787-141787msec

Disk stats (read/write):
  vdc: ios=523946/3, merge=0/1, ticks=243086/1, in_queue=243008, util=89.18%
```