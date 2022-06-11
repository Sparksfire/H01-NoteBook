## 理解Namespace

进去一个容器：

```bash
[root@VM-20-13-centos ~]# docker exec -it cnlanguage /bin/bash
root@cnlanguage:/var/www/html# ps
  PID TTY          TIME CMD
 2102 pts/1    00:00:00 bash
 2108 pts/1    00:00:00 ps
root@cnlanguage:/var/www/html# 
```
通过ps命令查看容器内部的进程，容器内部只有2个进程在运行，看不到操作系统的其他进程，一个进程执行`/bin/bash`，另一个执行`ps`，说明此刻容器被Docker隔离在了一个跟宿主机不同的环境中。

对于宿主机来说，我们就是在宿主机上执行了一个`/bin/bash`的进程，，宿主机给这个进程起了一个名字，比如PID=601，用来区分与其他进程的不同，Docker在运行`/bin/bash`的时候，会与宿主机上的其他程序进行隔离，让他看不到其他进程运行状况，并且重新计算进程号，也就是我们看到2102，但是这个进程实际上是宿主机的进程，这种技术，其实就是对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，这就是Linux里面的Namespace机制，也就是Docker采用的隔离技术。

Namespace是Linux内核用来隔离内核资源的方式。通过Namespace可以让一些进程只能看到与自己相关的一部分资源，而另外一些进程也只能看到与它们自己相关的资源，这2个进程根本就感觉不到对方的存在。具体的实现方式是把一个或多个进程的相关资源指定在同一个Namespace中。Linux Namespace是对全局系统资源的一种封装隔离，使得处于不同Namespace的进程拥有独立的全局系统资源，改变一个Namespace中的系统资源只会影响当前Namespace里的进程，对其他Namespace中的进程没有影响。

## Namespace类型介绍

目前，Linux 内核实现了6种 Namespace:

| Namespace | 作用 |
| :-----  | :---- |
| IPC     | 隔离System V IPC和POSIX消息队列 |
| NetWork | 隔离网络资源 |
| Mount   | 隔离文件系统挂载点 |
| PID     | 隔离进程ID |
| UTS     | 隔离主机名和域名 |
| User    | 隔离用户和用户组 |

## 六种命名空间

### 1.UTS Namespace

> UTS Namespace对主机名和域名进行隔离。那么为什么要隔离主机名呢？因为主机名可以代替IP地址来访问，如果不隔离，同名访问会出现冲突。简单地说，UTS namespace让容器有自己的hostname。默认情况下，容器的hostname是它的短ID，可以通过 -h或 --hostname参数设置。


### 2.IPC Namespace

> Linux提供了很多种进程通信机制，IPC Namespace针对System V和POSIX消息队列，这些IPC机制会使用标识符来区别不同的消息队列，然后两个进程通过标识符找到对应的消息队列。IPC Namespace使得相同的标识符在两个Namespace代表不同的消息队列，因此Namespace中的进程不能通过IPC来通信。

### 3.PID Namespace

> PID Namespace用来隔离进程的PID空间，使得不同PID Namespace里面的进程PID可以重复且互不影响。PID Namespace对容器累应用特别重要，可以是先容器内进程的暂停/恢复等功能，还可以支持容器在跨主机的迁移前后保持内部进行PID不发生变化。

### 4.Mount Namespace

> Mount Namespace 为进程提供独立的文件系统视图。可以这么理解，Mount Namespace 用来隔离文件系统的挂载点，这样进程就只能看到自己的Mount Namespace中的文件系统挂载点。进程的Mount Namespace中的挂载点信息可以在 /proc/[pid]/mounts、/proc/[pid]/mountinfo 和 /proc/[pid]/mountstats 这三个文件中找到。在一个 Namespace 里挂载、卸载的动作不会影响到其他 Namespace。

### 5.Network Namespace

> Network Namespace 在逻辑上是网络堆栈的一个副本，它有自己的路由、防火墙规则和网络设备。默认情况下，子进程继承其父进程的 Network Namespace。每个新创建的 Network Namespace 默认有一个本地环回接口 lo，除此之外，所有的其他网络设备(物理/虚拟网络接口，网桥等)只能属于一个 Network Namespace。每个 socket 也只能属于一个 Network Namespace。

### 6.User Namespace

> User Namespace 用于隔离安全相关的资源，包括 user IDs and group IDs，keys, 和 capabilities。同样一个用户的 user ID 和 group ID 在不同的User Namespace 中可以不一样(与 PID Namespace 类似)。可以这样理解，一个用户可以在一个User Namespace中是普通用户，但在另一个User Namespace中是root用户。


