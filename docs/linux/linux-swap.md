先来简单说说，swap的功能就是在对服务器物理内存不足的情况下造成的内存扩展记录的功能。所以当物理内存使用到一定量的情况下就会用到swap分区。

无论是Linux还是Windows系统 ，除了物理内存外还有一个虚拟内存。在Linux中，虚拟内存又被称为swap space。

那么我们一般在初始化系统的时候对于swap设置多少比较合适呢？

首先我们来看看Redhat官方文档中关于swap分区大小设置的建议。<a  href ="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/installation_guide/">官方文档</a>

![swap](../_images/linux-swap.png)

总结起来就是，如果不打算开启休眠功能，物理内存在8G以下，则swap设置为与物理内存一样大。如果物理内存在8G以上，swap空间设置为8G即可。当物理内存大于64G时，不建议开启休眠功能。

那么系统什么时候才会使用swap？

→交换分区主要是内存不够用的时候，将部分内存上的数据交换到swap空间上，以便让系统不会因为内存不够而导致oom或者更致命的情况出现。如果物理内存不够大，通过设置swap可以在内存不够用的时候不至于出发oom-killer导致某些关键进程被干掉。

→有些业务系统，如Redis，elasticsearch等，主要使用物理内存的系统，不希望它使用swap，因为大量使用swap会导致性能急剧下降。不设置swap的话，如果使用内存增加，那么可能会出现oom-killer的情况；如果设置了swap，此时可以通过设置/proc/sys/vm/swappiness，调整使用swap的概率，值越小，使用swap的概率就越低。

接下来我们在系统中来看一下：
```bash
# Ubuntu 20.04.4 LTS
ubuntu@i-o0g9eelb:~$ cat /proc/sys/vm/swappiness
60
ubuntu@i-o0g9eelb:~$ 

# centos 7.6
[root@VM-20-13-centos ~]# cat /proc/sys/vm/swappiness
30
[root@VM-20-13-centos ~]# 
```
swap的优化

swappiness值的大小对如何使用swap分区有着很大的联系，swappiness=0的时候表示最大限度的使用物理内存，然后才是swap空间；swappiness=100的时候表示积极使用swap分区，并且把内存上的数据及时搬运到swap空间里面。

按照Ubuntu 20.04.4 LTS下的默认值60，意思就是说，系统的物理内存在使用到100-60=40%的时候，就可以开始使用swap分区了 。此参数设置了使用交换分区的可能性大小。临时调整的方法如下，例如调整为10：
```bash
sysctl vm.swappiness=10
```

临时调的参数，会在系统重启之后恢复默认。想要永久调整的话，需要在/etc/sysctl.conf中修改，添加如下内容：
```bash
vm.swappiness=10

# 下面的方法也可以
cat >> /etc/sysctl.conf << EOF
vm.swappiness=10
EOF
```
这样配置完成后，表示在物理内存使用90%的时候，才考虑开始使用Swap，很明显，这样一来，对应用使用物理内存的概率大大增加了。在Redis、Kafka、Elasticsearch等内存型的应用中，可以考虑降低Swap的使用，通过优化swappiness，可以让系统最大限度使用物理内存而不是用Swap，提高应用系统性能。

结束，告辞！