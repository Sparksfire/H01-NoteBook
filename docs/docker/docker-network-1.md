Docker安装时会自动在host上创建三个网络，下面我们使用`docker network ls`命令来查看。

```bash
ubuntu@ubuntu:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
8d9e6dd9102a   bridge    bridge    local
e96dc150a8aa   host      host      local
48ac45c2d8d0   none      null      local
ubuntu@ubuntu:~$ 
```

## none

如果容器启动的时候使用`none`模式，那么容器中只有`lo`回环地址，没有其他任何网卡。这种类型的网络没有办法联网。容器创建时，可以通过 `--network=none`指定使用`none`网络

```bash
ubuntu@ubuntu:~$ docker run -it --network=none busybox
/ # 
/ # ifconfig
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # 
```

## host

如果容器启动的时候使用host模式，那么容器直接与宿主机使用同一网络命名空间，容器可以直接使用宿主机的ip地址和外面进行通信，如果宿主机具有公网ip地址时，那么容器也会拥有这个公有的IP地址，与此同时，容器内服务的端口地址也能直接使用宿主机的端口，无需再进行额外的NAT转换。因此host最大的优势就是网络性能比较好，但是容器将不再拥有隔离、独立的网络栈。

连接到host网络的容器共享Docker host的网络栈，容器的网络配置与host完全一样。可以通过 `--network=host`指定使用`host`网络，下面我们来看一下：

```bash
ubuntu@ubuntu:~$ docker run -it --network=host busybox
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel qlen 1000
    link/ether 00:0c:29:2e:f2:ba brd ff:ff:ff:ff:ff:ff
    inet 192.168.35.203/24 brd 192.168.35.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe2e:f2ba/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue 
    link/ether 02:42:61:c9:27:0a brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
/ # 
```
---
```bash
ubuntu@ubuntu:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:2e:f2:ba brd ff:ff:ff:ff:ff:ff
    inet 192.168.35.203/24 brd 192.168.35.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe2e:f2ba/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:61:c9:27:0a brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
ubuntu@ubuntu:~$ 
```

## bridge

当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，在此宿主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

容器启动的时候会从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0的网卡，另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中，到这里我们只是完成了宿主机与容器，同一主机上容器间的通信，此时容器还不能与外部网络进行通信。

为了使外界可以访问容器中的进程，docker采用了端口绑定的方式，也就是通过iptables的NAT，将宿主机上的端口流量转发到容器内的端口上。当使用使用docker run -p时，就是在iptables做了DNAT规则，实现端口转发功能。可以使用iptables -t nat -vnL查看。

![](https://images.icodedream.com/images/2022/05/15/docker-network-v1.jpg)

bridge模式中的容器与外界通信时，必定会占用宿主机上的端口，从而与宿主机竞争端口资源，此外由于容器与外界通信是基于三层iptables NAT，性能和效率上的损耗肯定是不可避免的。

![](https://images.icodedream.com/images/2022/05/15/docker-network-v4.jpg)

![](https://images.icodedream.com/images/2022/05/15/docker-network-v2.jpg)


如果在在启动容器前不指定--network,那么创建的容器将默认会挂到docker0上。

```bash
ubuntu@ubuntu:~$ brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024261c9270a	no		
ubuntu@ubuntu:~$ 
```

从上面的信息可以看出，当前docker0上没有任何其他网络设备，接下来我们创建一个容器看看。

```bash
ubuntu@ubuntu:~$ docker run -it busybox
/ # 

```

---

```bash
ubuntu@ubuntu:~$ brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024261c9270a	no		vethf77e117
ubuntu@ubuntu:~$ 
```

从上面可以看出一个名为vethf77e117被挂到了docker0上，vethf77e117就是新创建容器的虚拟网卡。下面观察一下容器的网络配置。

```bash
ubuntu@ubuntu:~$ docker run -it busybox
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 
```

容器有一个veth pair，那么为什么不是vethf77e117呢？实际上eth0@if13和vethf77e117是一对veth pair。veth pair是一种成对出现的特殊网络设备，可以他们想象成一根虚拟网线连接起来的一对网卡，一头是（eth0@if13）在容器中，另一头（vethf77e117）挂在docker0上。其效果就是将eth0@if13也挂在了docker0上。

我们从上面的信息中看到eth0@if13配置的ip是172.17.0.2，接下来我们通过dockernetwork inspect bridge查看一下bridge网络的配置信息。

```bash
ubuntu@ubuntu:~$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "8d9e6dd9102ad8d94d5b0cdf9428d4eb973279b27ee5d78402af734330919321",
        "Created": "2022-05-14T14:51:34.830635043Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "c31af13f01e5b140df79843298269aff7ae79405b3950e58e99344a12b09c3f0": {
                "Name": "affectionate_jones",
                "EndpointID": "6d73dabf7d417c0b3bf438db00bd98994df3111dfd2307724cc63473487eeee8",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
ubuntu@ubuntu:~$ 
```

那么从上面的信息中可以看出，bridge网络配置的subnet是172.17.0.0/16，网关172.17.0.1，那么这个网关在哪里呢？其实就是docker0

所以当前网络拓扑图如下：

```bash
ubuntu@ubuntu:~$ ifconfig docker0
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:61ff:fec9:270a  prefixlen 64  scopeid 0x20<link>
        ether 02:42:61:c9:27:0a  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 11  bytes 1186 (1.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ubuntu@ubuntu:~$ 
```
![](https://images.icodedream.com/images/2022/05/15/docker-network-v3.jpg)

容器创建时，docker会自动从172.17.0.0/16中分配一个IP，这里16位的掩码保证有足够多的IP可以供容器使用

## container

container模式，这个模式指定新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。


## user-defined

除了none、host、bridge这三个自动创建的网络，用户也可以根据业务需要创建user-defined网络。

在实际的业务总，我们还可以通过bridge驱动创建类似前面默认的bridge网络。

```bash
ubuntu@ubuntu:~$ docker network create --driver bridge mynet
9a9d5396c7db6aacaf6186d59c6511967e556fcb18d56b03767a91b07a1b822b
ubuntu@ubuntu:~$ brctl show
bridge name	bridge id		STP enabled	interfaces
br-9a9d5396c7db		8000.02421523d388	no		
docker0		8000.024261c9270a	no		
ubuntu@ubuntu:~$ ifconfig br-9a9d5396c7db
br-9a9d5396c7db: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:15:23:d3:88  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ubuntu@ubuntu:~$ 
```

下面查看一下当前host网络的结构变化

```bash
ubuntu@ubuntu:~$ docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "9a9d5396c7db6aacaf6186d59c6511967e556fcb18d56b03767a91b07a1b822b",
        "Created": "2022-05-15T08:54:15.59250957Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
ubuntu@ubuntu:~$ 
```

那么我们可以自定义ip网段吗？当然可以，我们只需要在创建网络的时候指定--subnet和--geteway参数。

```bash
ubuntu@ubuntu:~$ docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 mynet2
ac4d812a689e0895b9873f42ecfb9181a64fb85e183b630c91ec37bb2b575654
ubuntu@ubuntu:~$ docker network inspect mynet2
[
    {
        "Name": "mynet2",
        "Id": "ac4d812a689e0895b9873f42ecfb9181a64fb85e183b630c91ec37bb2b575654",
        "Created": "2022-05-15T08:59:11.446186522Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.22.16.0/24",
                    "Gateway": "172.22.16.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
ubuntu@ubuntu:~$ brctl show
bridge name	bridge id		STP enabled	interfaces
br-9a9d5396c7db		8000.02421523d388	no		
br-ac4d812a689e		8000.0242f74d2238	no		
docker0		8000.024261c9270a	no		
ubuntu@ubuntu:~$ ifconfig br-ac4d812a689e
br-ac4d812a689e: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.22.16.1  netmask 255.255.255.0  broadcast 172.22.16.255
        ether 02:42:f7:4d:22:38  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ubuntu@ubuntu:~$ 
```

那么容器在启动的时候，我们需要通过--network参数指定创建的网络。这样容器的ip都是docker自动从对应的subnet中分配。当然了，我们还可以通过--ip指定静态ip。
```bash
ubuntu@ubuntu:~$ docker run -it --network=mynet2 busybox
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:16:10:02 brd ff:ff:ff:ff:ff:ff
    inet 172.22.16.2/24 brd 172.22.16.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 
```

---

```bash
ubuntu@ubuntu:~$ docker run -it --network=mynet2 --ip 172.22.16.10 busybox
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
20: eth0@if21: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:16:10:0a brd ff:ff:ff:ff:ff:ff
    inet 172.22.16.10/24 brd 172.22.16.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 
```

!> 注：只有使用 --subnet创建的网络才能指定静态IP