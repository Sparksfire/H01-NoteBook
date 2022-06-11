在升级系统内核前，我们先来了解一下什么事内核（Kernel）？


内核是操作系统的关键组件。 它借助进程间通信和系统调用，在硬件级别上充当应用程序和数据处理之间的桥梁。
每当将操作系统加载到内存中时，首先，将加载内核并将其保留在那里，直到操作系统关闭。 内核负责处理低级任务，例如任务管理，内存管理，风险管理等。
Linux内核[kernel]是整个操作系统的最底层，它负责整个硬件的驱动，以及提供各种系统所需的核心功能，包括防火墙机制、是否支持LVM或Quota等文件系统等等，如果内核不认识某个最新的硬件，那么硬件也就无法被驱动，你也就无法使用该硬件。


## 查看当前系统内核版本

```bash
[root@localhost ~]# uname -srm
Linux 3.10.0-957.el7.x86_64 x86_64
[root@localhost ~]# 
```

## 添加yum源

```bash
[root@localhost ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
[root@localhost ~]# rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
[root@localhost ~]# 
```

## 更新内核

```bash
# 查看最新内核版本
[root@localhost ~]# yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * elrepo-kernel: hkg.mirror.rackspace.com
elrepo-kernel                                                                                                                                                                                                       | 3.0 kB  00:00:00     
elrepo-kernel/primary_db                                                                                                                                                                                            | 2.1 MB  00:00:00     
Available Packages
elrepo-release.noarch                                                                                                7.0-5.el7.elrepo                                                                                         elrepo-kernel
kernel-lt.x86_64                                                                                                     5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-lt-devel.x86_64                                                                                               5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-lt-doc.noarch                                                                                                 5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-lt-headers.x86_64                                                                                             5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-lt-tools.x86_64                                                                                               5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-lt-tools-libs.x86_64                                                                                          5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-lt-tools-libs-devel.x86_64                                                                                    5.4.188-1.el7.elrepo                                                                                     elrepo-kernel
kernel-ml.x86_64                                                                                                     5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
kernel-ml-devel.x86_64                                                                                               5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
kernel-ml-doc.noarch                                                                                                 5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
kernel-ml-headers.x86_64                                                                                             5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
kernel-ml-tools.x86_64                                                                                               5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
kernel-ml-tools-libs.x86_64                                                                                          5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
kernel-ml-tools-libs-devel.x86_64                                                                                    5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
perf.x86_64                                                                                                          5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
python-perf.x86_64                                                                                                   5.17.2-1.el7.elrepo                                                                                      elrepo-kernel
[root@localhost ~]# 
# 安装内核
[root@localhost ~]# yum --disablerepo="*" --enablerepo="elrepo-kernel" install -y kernel-ml-5.17.2-1.el7.elrepo
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * elrepo-kernel: hkg.mirror.rackspace.com
Resolving Dependencies
--> Running transaction check
---> Package kernel-ml.x86_64 0:5.17.2-1.el7.elrepo will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================================================================================================================================================
 Package                                               Arch                                               Version                                                          Repository                                                 Size
===========================================================================================================================================================================================================================================
Installing:
 kernel-ml                                             x86_64                                             5.17.2-1.el7.elrepo                                              elrepo-kernel                                              56 M

Transaction Summary
===========================================================================================================================================================================================================================================
Install  1 Package

Total download size: 56 M
Installed size: 255 M
Downloading packages:
kernel-ml-5.17.2-1.el7.elrepo.x86_64.rpm                                                                                                                                                                            |  56 MB  00:00:05     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : kernel-ml-5.17.2-1.el7.elrepo.x86_64                                                                                                                                                                                    1/1 
  Verifying  : kernel-ml-5.17.2-1.el7.elrepo.x86_64                                                                                                                                                                                    1/1 

Installed:
  kernel-ml.x86_64 0:5.17.2-1.el7.elrepo                                                                                                                                                                                                   

Complete!
[root@localhost ~]#
```

## 设置内核启动

```
# 查看所有内核
[root@localhost ~]# sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.17.2-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
2 : CentOS Linux (0-rescue-2acfe84229a64f0c8669ff92dcfeea8d) 7 (Core)
[root@localhost ~]# 
# 设置采用最新内核
[root@localhost ~]# sudo grub2-set-default 0
# 重启系统
[root@localhost ~]# reboot
# 验证
[root@localhost ~]# uname -srm
Linux 5.17.2-1.el7.elrepo.x86_64 x86_64
[root@localhost ~]# 
```

结束，告辞！

