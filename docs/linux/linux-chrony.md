
本教程基于Centos 7.6版本。

[Chrony官方文档](https://chrony.tuxfamily.org/)
> `chrony` is a versatile implementation of the Network Time Protocol (NTP). It can synchronise the system clock with NTP servers, reference clocks (e.g. GPS receiver), and manual input using wristwatch and keyboard. It can also operate as an NTPv4 (RFC 5905) server and peer to provide a time service to other computers in the network.
>
> It is designed to perform well in a wide range of conditions, including intermittent network connections, heavily congested networks, changing temperatures (ordinary computer clocks are sensitive to temperature), and systems that do not run continuosly, or run on a virtual machine.
>
> Typical accuracy between two machines synchronised over the Internet is within a few milliseconds; on a LAN, accuracy is typically in tens of microseconds. With hardware timestamping, or a hardware reference clock, sub-microsecond accuracy may be possible.
>
> Two programs are included in `chrony`, `chronyd` is a daemon that can be started at boot time and `chronyc` is a command-line interface program which can be used to monitor `chronyd`’s performance and to change various operating parameters whilst it is running.

## 主机规划

| 192.168.35.201 | 服务端 |
| --- | --- |
| 192.168.35.202 | 客户端 |

>[!Tip] 在安装Chrony前，我们需要设置一下时区： `timedatectl set-timezone Asia/Shanghai
`


## 安装Chrony

```bash
# 2台设备都需要安装
[root@localhost ~]# yum -y install chrony
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package chrony.x86_64 0:3.4-1.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================================================================================================================================================
 Package                                                 Arch                                                    Version                                                       Repository                                             Size
===========================================================================================================================================================================================================================================
Installing:
 chrony                                                  x86_64                                                  3.4-1.el7                                                     base                                                  251 k

Transaction Summary
===========================================================================================================================================================================================================================================
Install  1 Package

Total download size: 251 k
Installed size: 491 k
Downloading packages:
chrony-3.4-1.el7.x86_64.rpm                                                                                                                                                                                         | 251 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : chrony-3.4-1.el7.x86_64                                                                                                                                                                                                 1/1 
  Verifying  : chrony-3.4-1.el7.x86_64                                                                                                                                                                                                 1/1 

Installed:
  chrony.x86_64 0:3.4-1.el7                                                                                                                                                                                                                

Complete!
[root@localhost ~]# 
```

## Chrony配置文件

`/etc/chrony.conf`

```bash
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
# 配置NTP服务器
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
# 记录系统时钟获得/丢失时间的速率至drift文件中
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
# 
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
# 启用RTC（实时时钟）的内核同步
rtcsync

# Enable hardware timestamping on all interfaces that support it.
# 通过使用 hwtimestamp 指令启用硬件时间戳
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
# 指定NTP客户端地址，以允许或拒绝连接到时钟服务器的机器
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
# NTP服务器不可用时，采用本地时间作为同步标准
#local stratum 10

# Specify file containing keys for NTP authentication.
# 指定包含NTP验证密钥的文件
#keyfile /etc/chrony.keys

# Specify directory for log files.
# 日志目录
logdir /var/log/chrony

# Select which information is logged.
# 选择日志文件要记录的信息
#log measurements statistics tracking
```

## 服务端配置

如果服务器无法连接外网，则可以设置为`127.0.0.1`，向内网`192.168.35.0`网段的服务器提供时钟服务。如果不限制IP段，则可以设置设置为`allow all`

```bash
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.asia.pool.ntp.org iburst
server 1.asia.pool.ntp.org iburst
server 2.asia.pool.ntp.org iburst
server 3.asia.pool.ntp.org iburst

# Allow NTP client access from local network.
allow 192.168.35.0/16
```

### 设置开机启动chronyd服务并重启chronyd服务

```bash
systemctl enable chronyd && systemctl restart chronyd

[root@localhost ~]# systemctl enable chronyd && systemctl restart chronyd
[root@localhost ~]# systemctl status chronyd
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2022-04-18 18:06:06 CST; 16s ago
     Docs: man:chronyd(8)
           man:chrony.conf(5)
  Process: 1789 ExecStartPost=/usr/libexec/chrony-helper update-daemon (code=exited, status=0/SUCCESS)
  Process: 1785 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 1787 (chronyd)
    Tasks: 1
   Memory: 376.0K
   CGroup: /system.slice/chronyd.service
           └─1787 /usr/sbin/chronyd

Apr 18 18:06:06 localhost.localdomain systemd[1]: Starting NTP client/server...
Apr 18 18:06:06 localhost.localdomain chronyd[1787]: chronyd version 3.4 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP +SCFILTER +SIGND +ASYNCDNS +SECHASH +IPV6 +DEBUG)
Apr 18 18:06:06 localhost.localdomain systemd[1]: Started NTP client/server.
Apr 18 18:06:12 localhost.localdomain chronyd[1787]: Selected source 202.118.1.81
Apr 18 18:06:12 localhost.localdomain chronyd[1787]: System clock wrong by 1.527508 seconds, adjustment started
Apr 18 18:06:14 localhost.localdomain chronyd[1787]: System clock was stepped by 1.527508 seconds
[root@localhost ~]#

# 查看监听状态
[root@localhost ~]# netstat -apn | grep chrony
udp        0      0 127.0.0.1:323           0.0.0.0:*                           1787/chronyd        
udp        0      0 0.0.0.0:123             0.0.0.0:*                           1787/chronyd        
udp6       0      0 ::1:323                 :::*                                1787/chronyd        
unix  2      [ ]         DGRAM                    33048    1787/chronyd         /var/run/chrony/chronyd.sock
unix  2      [ ]         DGRAM                    34427    1787/chronyd         
[root@localhost ~]#

# 开启网络时间同步功能
[root@localhost ~]# timedatectl set-ntp true
[root@localhost ~]# 

# NTP synchronized: yes，表示时钟同步成功
[root@localhost ~]# timedatectl
      Local time: Mon 2022-04-18 18:09:12 CST
  Universal time: Mon 2022-04-18 10:09:12 UTC
        RTC time: Mon 2022-04-18 10:09:12
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
[root@localhost ~]#

# ^*开头的行，说明已经与服务器时间同步
[root@localhost ~]# chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* time.neu.edu.cn               1  10   377   675  +1573us[+1844us] +/-   32ms
^- time1.isu.net.sa              2  10   367  1174    +15ms[  +15ms] +/-  345ms
^+ 103.110.86.52                 2  10   357   884   -882us[ -621us] +/-   67ms
^- time3.unima.ac.id             2  10    17   870  +2460us[+2722us] +/-  305ms
[root@localhost ~]# 
```

## 配置客户端

前面我们已经在客户端安装了Chrony，接下来只需要修改以下配置文件和启动服务即可。在客户端指定192.168.35.201节点为上游NTP服务器

```bash
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 192.168.35.201 iburst
```

### 设置开机启动chronyd服务并重启chronyd服务

```bash
[root@localhost ~]# systemctl enable chronyd && systemctl restart chronyd
[root@localhost ~]# systemctl status chronyd
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2022-04-18 18:17:31 CST; 1min 0s ago
     Docs: man:chronyd(8)
           man:chrony.conf(5)
  Process: 2198 ExecStartPost=/usr/libexec/chrony-helper update-daemon (code=exited, status=0/SUCCESS)
  Process: 2195 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 2197 (chronyd)
    Tasks: 1
   Memory: 280.0K
   CGroup: /system.slice/chronyd.service
           └─2197 /usr/sbin/chronyd

Apr 18 18:17:31 localhost.localdomain systemd[1]: Starting NTP client/server...
Apr 18 18:17:31 localhost.localdomain chronyd[2197]: chronyd version 3.4 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP +SCFILTER +SIGND +ASYNCDNS +SECHASH +IPV6 +DEBUG)
Apr 18 18:17:31 localhost.localdomain systemd[1]: Started NTP client/server.
[root@localhost ~]# 
```

执行以下命令，我们可以看到`NTP synchronized: yes`，说明已经开始同步。还可以在执行`chronyc sources`命令后可以看到其中`^*`中的`*`表示从该服务器进行了时钟同步，如果显示为`^?`则说明没有从该服务器同步时钟。

```bash
[root@localhost ~]# timedatectl
      Local time: Mon 2022-04-18 18:22:37 CST
  Universal time: Mon 2022-04-18 10:22:37 UTC
        RTC time: Mon 2022-04-18 10:22:37
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
[root@localhost ~]# chronyc sources -v
210 Number of sources = 1

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 192.168.35.201                2   6   377    25    -23us[  -40us] +/-   33ms
[root@localhost ~]# 
```

>[!Tip]Chrony运行于UDP323端口，ntp运行于UDP123端口，使用chrony服务器可以同时为chrony客户端和ntp客户端提供服务。如果出现没有从服务端同步成功的情况，请检查防火墙。

## Chrony的一些命令：

### 查看ntp_servers

```bash
[root@localhost ~]# chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* time.neu.edu.cn               1  10   377   847  +1573us[+1844us] +/-   32ms
^- time1.isu.net.sa              2  10   367   22m    +15ms[  +15ms] +/-  345ms
^+ 103.110.86.52                 2  10   337    14   -611us[ -611us] +/-   69ms
^- time3.unima.ac.id             2  10    37     1  -7646us[-7646us] +/-  311ms
[root@localhost ~]# 
```

### 查看 ntp_servers 状态

```bash
[root@localhost ~]# chronyc sourcestats -v
210 Number of sources = 4
                             .- Number of sample points in measurement set.
                            /    .- Number of residual runs with same sign.
                           |    /    .- Length of measurement set (time).
                           |   |    /      .- Est. clock freq error (ppm).
                           |   |   |      /           .- Est. error in freq.
                           |   |   |     |           /         .- Est. offset.
                           |   |   |     |          |          |   On the -.
                           |   |   |     |          |          |   samples. \
                           |   |   |     |          |          |             |
Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
==============================================================================
time.neu.edu.cn            27  16   98m     +0.009      0.117   +803us   291us
time1.isu.net.sa           25  14   89m     -0.597      2.087    -18ms  3174us
103.110.86.52              31  17  112m     -0.044      0.966  -1570us  2423us
time3.unima.ac.id          28  18  111m     -0.662      2.682  -3931us  5460us
[root@localhost ~]# 
```

### 查看 ntp_servers 是否在线

```bash
[root@localhost ~]# chronyc activity -v
200 OK
4 sources online
0 sources offline
0 sources doing burst (return to online)
0 sources doing burst (return to offline)
0 sources with unknown address
[root@localhost ~]# 
```

### 查看 ntp 详细信息

```bash
[root@localhost ~]# chronyc tracking -v
Reference ID    : CA760151 (time.neu.edu.cn)
Stratum         : 2
Ref time (UTC)  : Mon Apr 18 11:44:19 2022
System time     : 0.000445690 seconds fast of NTP time
Last offset     : +0.000270649 seconds
RMS offset      : 0.000712963 seconds
Frequency       : 23.932 ppm slow
Residual freq   : +0.008 ppm
Skew            : 0.141 ppm
Root delay      : 0.063534558 seconds
Root dispersion : 0.001258016 seconds
Update interval : 1032.4 seconds
Leap status     : Normal
[root@localhost ~]# 
```

### 强制同步系统时钟

```bash
[root@localhost ~]# chronyc -a makestep
200 OK
[root@localhost ~]# 
```

结束，告辞！