
在Centos 7使用yum命令安装Docker的前提是必须配置Docker YUM源。安装Docker的系统内核版本不能低于3.10。否则部分功能将无法使用，如overlay2存储层驱动，而且部分功能还可能不太稳定。

如需升级内核，可以参考文章：[Centos 7升级Linux内核版本](https://icodedream.com/post/linux/Centos-7%E5%8D%87%E7%BA%A7%E5%86%85%E6%A0%B8%E7%89%88%E6%9C%AC/)
查看内核系统内核版本

```bash
[root@localhost ~]# uname -srm
Linux 3.10.0-862.el7.x86_64 x86_64
[root@localhost ~]# 
```

## 卸载旧版本

```bash
yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-selinux \
           docker-engine-selinux \
           docker-engine
```

```bash
[root@localhost ~]# yum remove docker \
>            docker-client \
>            docker-client-latest \
>            docker-common \
>            docker-latest \
>            docker-latest-logrotate \
>            docker-logrotate \
>            docker-selinux \
>            docker-engine-selinux \
>            docker-engine
Loaded plugins: fastestmirror
No Match for argument: docker
No Match for argument: docker-client
No Match for argument: docker-client-latest
No Match for argument: docker-common
No Match for argument: docker-latest
No Match for argument: docker-latest-logrotate
No Match for argument: docker-logrotate
No Match for argument: docker-selinux
No Match for argument: docker-engine-selinux
No Match for argument: docker-engine
No Packages marked for removal
[root@localhost ~]# 
```

## 使用YUM安装
  ### 安装所需要的依赖

```bash
[root@localhost ~]# yum install -y yum-utils
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                                                                                                                                                                | 3.6 kB  00:00:00     
extras                                                                                                                                                                                                              | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                             | 2.9 kB  00:00:00     
(1/4): base/7/x86_64/primary_db                                                                                                                                                                                     | 6.1 MB  00:00:00     
(2/4): base/7/x86_64/group_gz                                                                                                                                                                                       | 153 kB  00:00:00     
(3/4): extras/7/x86_64/primary_db                                                                                                                                                                                   | 246 kB  00:00:01     
(4/4): updates/7/x86_64/primary_db                                                                                                                                                                                  |  15 MB  00:00:58     
Resolving Dependencies
--> Running transaction check
---> Package yum-utils.noarch 0:1.1.31-54.el7_8 will be installed
--> Processing Dependency: python-kitchen for package: yum-utils-1.1.31-54.el7_8.noarch
--> Processing Dependency: libxml2-python for package: yum-utils-1.1.31-54.el7_8.noarch
--> Running transaction check
---> Package libxml2-python.x86_64 0:2.9.1-6.el7_9.6 will be installed
--> Processing Dependency: libxml2 = 2.9.1-6.el7_9.6 for package: libxml2-python-2.9.1-6.el7_9.6.x86_64
---> Package python-kitchen.noarch 0:1.1.1-5.el7 will be installed
--> Processing Dependency: python-chardet for package: python-kitchen-1.1.1-5.el7.noarch
--> Running transaction check
---> Package libxml2.x86_64 0:2.9.1-6.el7_2.3 will be updated
---> Package libxml2.x86_64 0:2.9.1-6.el7_9.6 will be an update
---> Package python-chardet.noarch 0:2.2.1-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================================================================================================================================================
 Package                                                     Arch                                                Version                                                        Repository                                            Size
===========================================================================================================================================================================================================================================
Installing:
 yum-utils                                                   noarch                                              1.1.31-54.el7_8                                                base                                                 122 k
Installing for dependencies:
 libxml2-python                                              x86_64                                              2.9.1-6.el7_9.6                                                updates                                              247 k
 python-chardet                                              noarch                                              2.2.1-3.el7                                                    base                                                 227 k
 python-kitchen                                              noarch                                              1.1.1-5.el7                                                    base                                                 267 k
Updating for dependencies:
 libxml2                                                     x86_64                                              2.9.1-6.el7_9.6                                                updates                                              668 k

Transaction Summary
===========================================================================================================================================================================================================================================
Install  1 Package  (+3 Dependent packages)
Upgrade             ( 1 Dependent package)

Total download size: 1.5 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
warning: /var/cache/yum/x86_64/7/base/packages/python-chardet-2.2.1-3.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for python-chardet-2.2.1-3.el7.noarch.rpm is not installed
(1/5): python-chardet-2.2.1-3.el7.noarch.rpm                                                                                                                                                                        | 227 kB  00:00:00     
(2/5): python-kitchen-1.1.1-5.el7.noarch.rpm                                                                                                                                                                        | 267 kB  00:00:00     
Public key for libxml2-python-2.9.1-6.el7_9.6.x86_64.rpm is not installed
(3/5): libxml2-python-2.9.1-6.el7_9.6.x86_64.rpm                                                                                                                                                                    | 247 kB  00:00:00     
(4/5): yum-utils-1.1.31-54.el7_8.noarch.rpm                                                                                                                                                                         | 122 kB  00:00:00     
(5/5): libxml2-2.9.1-6.el7_9.6.x86_64.rpm                                                                                                                                                                           | 668 kB  00:00:00     
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                      3.4 MB/s | 1.5 MB  00:00:00     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-5.1804.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : libxml2-2.9.1-6.el7_9.6.x86_64                                                                                                                                                                                          1/6 
  Installing : libxml2-python-2.9.1-6.el7_9.6.x86_64                                                                                                                                                                                   2/6 
  Installing : python-chardet-2.2.1-3.el7.noarch                                                                                                                                                                                       3/6 
  Installing : python-kitchen-1.1.1-5.el7.noarch                                                                                                                                                                                       4/6 
  Installing : yum-utils-1.1.31-54.el7_8.noarch                                                                                                                                                                                        5/6 
  Cleanup    : libxml2-2.9.1-6.el7_2.3.x86_64                                                                                                                                                                                          6/6 
  Verifying  : python-chardet-2.2.1-3.el7.noarch                                                                                                                                                                                       1/6 
  Verifying  : libxml2-2.9.1-6.el7_9.6.x86_64                                                                                                                                                                                          2/6 
  Verifying  : libxml2-python-2.9.1-6.el7_9.6.x86_64                                                                                                                                                                                   3/6 
  Verifying  : python-kitchen-1.1.1-5.el7.noarch                                                                                                                                                                                       4/6 
  Verifying  : yum-utils-1.1.31-54.el7_8.noarch                                                                                                                                                                                        5/6 
  Verifying  : libxml2-2.9.1-6.el7_2.3.x86_64                                                                                                                                                                                          6/6 

Installed:
  yum-utils.noarch 0:1.1.31-54.el7_8                                                                                                                                                                                                       

Dependency Installed:
  libxml2-python.x86_64 0:2.9.1-6.el7_9.6                                         python-chardet.noarch 0:2.2.1-3.el7                                         python-kitchen.noarch 0:1.1.1-5.el7                                        

Dependency Updated:
  libxml2.x86_64 0:2.9.1-6.el7_9.6                                                                                                                                                                                                         

Complete!
[root@localhost ~]# 
```

### 添加Docker YUM源

```bash
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

[root@localhost ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[root@localhost ~]# 
```

### 更新yum软件包索引

```bash
yum makecache fast

[root@localhost ~]# yum makecache fast
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                                                                                                                                                                | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                                                                    | 3.5 kB  00:00:00     
extras                                                                                                                                                                                                              | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                             | 2.9 kB  00:00:00     
(1/2): docker-ce-stable/7/x86_64/updateinfo                                                                                                                                                                         |   55 B  00:00:00     
(2/2): docker-ce-stable/7/x86_64/primary_db                                                                                                                                                                         |  75 kB  00:00:00     
Metadata Cache Created
[root@localhost ~]# 
```

### 查看存储库中 Docker 的版本

```bash
yum list docker-ce --showduplicates | sort -r

[root@localhost ~]# yum list docker-ce --showduplicates | sort -r
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
 * extras: mirrors.aliyun.com
docker-ce.x86_64            3:20.10.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.14-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.13-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.12-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.11-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.10-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.15-3.el7                    docker-ce-stable
docker-ce.x86_64            3:19.03.14-3.el7                    docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.13-3.el7                    docker-ce-stable
docker-ce.x86_64            3:19.03.12-3.el7                    docker-ce-stable
docker-ce.x86_64            3:19.03.11-3.el7                    docker-ce-stable
docker-ce.x86_64            3:19.03.10-3.el7                    docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
 * base: mirrors.aliyun.com
Available Packages
[root@localhost ~]# 
```

可以从上面的信息中看出，当前最新稳定版本v20.10.14

### 安装最新版本可以使用下面的命令

```bash
yum install docker-ce docker-ce-cli containerd.io
```

### 安装特定版本的 Docker（v20.10.9）

```bash
yum -y install docker-ce-3:20.10.9-3.el7.x86_64 docker-ce-cli-3:20.10.9-3.el7.x86_64 containerd.io
```

## 使用脚本自动安装

在这里我们为了方便使用脚本自动安装部署，至于其他安装方法大家可以自行选择；

```bash
# 执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 的稳定(stable)版本安装在系统中
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh --mirror Aliyun
```

```bash
[root@localhost ~]# curl -fsSL get.docker.com -o get-docker.sh
[root@localhost ~]# ls -al
total 48
dr-xr-x---.  3 root root   168 Apr 18 13:24 .
dr-xr-xr-x. 17 root root   224 Jun 20  2021 ..
-rw-------.  1 root root  1243 Jun 20  2021 anaconda-ks.cfg
-rw-------.  1 root root    71 Jun 20  2021 .bash_history
-rw-r--r--.  1 root root    18 Dec 29  2013 .bash_logout
-rw-r--r--.  1 root root   176 Dec 29  2013 .bash_profile
-rw-r--r--.  1 root root   176 Dec 29  2013 .bashrc
-rw-r--r--.  1 root root   100 Dec 29  2013 .cshrc
-rw-r--r--.  1 root root 18617 Apr 18 13:24 get-docker.sh
drwxr-----.  3 root root    19 Apr 18 13:02 .pki
-rw-r--r--.  1 root root   129 Dec 29  2013 .tcshrc
[root@localhost ~]# sh get-docker.sh --mirror Aliyun 
# Executing docker install script, commit: 93d2499759296ac1f9c510605fef85052a2c32be
+ sh -c 'yum install -y -q yum-utils'
Package yum-utils-1.1.31-54.el7_8.noarch already installed and latest version
+ sh -c 'yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo'
Loaded plugins: fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
+ '[' stable '!=' stable ']'
+ sh -c 'yum makecache'
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                                                                                                                                                                | 3.6 kB  00:00:00     
docker-ce-stable                                                                                                                                                                                                    | 3.5 kB  00:00:00     
extras                                                                                                                                                                                                              | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                             | 2.9 kB  00:00:00     
(1/8): base/7/x86_64/other_db                                                                                                                                                                                       | 2.6 MB  00:00:00     
(2/8): extras/7/x86_64/filelists_db                                                                                                                                                                                 | 277 kB  00:00:00     
(3/8): base/7/x86_64/filelists_db                                                                                                                                                                                   | 7.2 MB  00:00:00     
(4/8): extras/7/x86_64/other_db                                                                                                                                                                                     | 147 kB  00:00:00     
(5/8): docker-ce-stable/7/x86_64/filelists_db                                                                                                                                                                       |  31 kB  00:00:00     
(6/8): updates/7/x86_64/other_db                                                                                                                                                                                    | 1.0 MB  00:00:00     
(7/8): docker-ce-stable/7/x86_64/other_db                                                                                                                                                                           | 123 kB  00:00:01     
(8/8): updates/7/x86_64/filelists_db                                                                                                                                                                                | 8.2 MB  00:00:04     
Metadata Cache Created
+ '[' -n '' ']'
+ sh -c 'yum install -y -q docker-ce'
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/containerd.io-1.5.11-3.1.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Public key for containerd.io-1.5.11-3.1.el7.x86_64.rpm is not installed
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
+ version_gte 20.10
+ '[' -z '' ']'
+ return 0
+ sh -c 'yum install -y -q docker-ce-rootless-extras'
Package docker-ce-rootless-extras-20.10.14-3.el7.x86_64 already installed and latest version

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================

[root@localhost ~]# 
```

## 启动Docker并设置开机启动

```bash
systemctl enable docker && systemctl start docker

[root@localhost ~]# systemctl enable docker && systemctl start docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@localhost ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-04-18 13:26:41 CST; 7min ago
     Docs: https://docs.docker.com
 Main PID: 11337 (dockerd)
    Tasks: 9
   Memory: 37.6M
   CGroup: /system.slice/docker.service
           └─11337 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Apr 18 13:26:40 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:40.666975867+08:00" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <ni...l>}" module=grpc
Apr 18 13:26:40 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:40.666984232+08:00" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
Apr 18 13:26:40 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:40.689880052+08:00" level=info msg="Loading containers: start."
Apr 18 13:26:41 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:41.419839037+08:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used ...rred IP address"
Apr 18 13:26:41 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:41.560042154+08:00" level=info msg="Firewalld: interface docker0 already part of docker zone, returning"
Apr 18 13:26:41 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:41.675548993+08:00" level=info msg="Loading containers: done."
Apr 18 13:26:41 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:41.695931301+08:00" level=info msg="Docker daemon" commit=87a90dc graphdriver(s)=overlay2 version=20.10.14
Apr 18 13:26:41 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:41.696316981+08:00" level=info msg="Daemon has completed initialization"
Apr 18 13:26:41 localhost.localdomain systemd[1]: Started Docker Application Container Engine.
Apr 18 13:26:41 localhost.localdomain dockerd[11337]: time="2022-04-18T13:26:41.722195313+08:00" level=info msg="API listen on /var/run/docker.sock"
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost ~]# 
```

### 验证 Docker 是否安装成功

```bash
docker version

[root@localhost ~]# docker version
Client: Docker Engine - Community
 Version:           20.10.14
 API version:       1.41
 Go version:        go1.16.15
 Git commit:        a224086
 Built:             Thu Mar 24 01:49:57 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.14
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.15
  Git commit:       87a90dc
  Built:            Thu Mar 24 01:48:24 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.5.11
  GitCommit:        3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc:
  Version:          1.0.3
  GitCommit:        v1.0.3-0-gf46b6ba
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
[root@localhost ~]# 
```

### 创建Docker用户组

默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

创建`Docker`组：

```bash
groupadd docker

[root@localhost ~]# groupadd docker
groupadd: group 'docker' already exists
[root@localhost ~]# 
```

将当前用户加入 `Docker` 组

```bash
usermod -aG docker $USER

[root@localhost ~]# usermod -aG docker $USER
[root@localhost ~]# 
```

退出当前终端并重新登录，当然也可以使用以下命令不退出终端和重新登录

```bash
newgrp docker

[root@localhost ~]# newgrp docker
[root@localhost ~]# cat /etc/group | grep docker
docker:x:995:root
[root@localhost ~]# 
```

> newgrp：如果用户 user1 有个多个用户组(初始组：user1，附加组：group1，group2)，则登录后默认的有效组是该用户的初始组user1，此时该用户创建的文件所属组就是 user1，如果让该用户创建的文件所属组为 group1，就需要使用 "newgrp" 命令切换有效组为 group1