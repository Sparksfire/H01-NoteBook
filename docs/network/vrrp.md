## 什么是VRRP？

VRRP-虚拟路由冗余协议（Virtual Router Redundancy Protocol）是一种用于提高网络可靠性的容错协议。通过VRRP，可以在主机的下一跳设备出现故障时，及时将业务切换到备份设备，从而保障网络通信的连续性和可靠性。

## VRRP的工作原理

### VRRP三种状态
VRRP协议中定义了三种状态：初始状态（Initialize）、活动状态（Master）、备份状态（Backup）。其中，只有处于Master状态的设备才可以转发那些发送到虚拟IP地址的报文。

VRRP协议状态：

| 状态 | 说明 |
| :--- | :--- |
| Initialize | 该状态为VRRP不可用状态，在此状态时设备不会对VRRP报文做任何处理。通常刚配置VRRP时或设备检测到故障时会进入Initialize状态。收到接口Up的消息后，如果设备的优先级为255，则直接成为Master设备；如果设备的优先级小于255，则会先切换至Backup状态。 |
| Master | 当VRRP设备处于Master状态时，它将会做下列工作：<br>• 定时（Advertisement Interval）发送VRRP通告报文。<br>• 以虚拟MAC地址响应对虚拟IP地址的ARP请求。<br>• 转发目的MAC地址为虚拟MAC地址的IP报文。<br>• 如果它是这个虚拟IP地址的拥有者，则接收目的IP地址为这个虚拟IP地址的IP报文。否则，丢弃这个IP报文。<br>• 如果收到比自己优先级大的报文，立即成为Backup。<br>• 如果收到与自己优先级相等的VRRP报文且本地接口IP地址小于对端接口IP，立即成为Backup。|
| Backup | 	当VRRP设备处于Backup状态时，它将会做下列工作：<br>• 接收Master设备发送的VRRP通告报文，判断Master设备的状态是否正常。<br>• 对虚拟IP地址的ARP请求，不做响应。<br>• 丢弃目的IP地址为虚拟IP地址的IP报文。<br>• 如果收到优先级和自己相同或者比自己大的报文，则重置Master_Down_Interval定时器，不进一步比较IP地址。Master_Down_Interval定时器：Backup设备在该定时器超时后仍未收到通告报文，则会转换为Master状态。计算公式如下：Master_Down_Interval=(3* Advertisement_Interval) + Skew_time。其中，Skew_Time=(256–Priority)/256。<br>• 如果收到比自己优先级小的报文且该报文优先级是0时，定时器时间设置为Skew_time（偏移时间），如果该报文优先级不是0，丢弃报文，立刻成为Master。|


### VRRP定时器

偏移时间（Skew_Time）是用来避免当Master出现故障时，备份组中会有多个Backup在同一时刻 同时转变为Master，导致备份组中存在多台Master。 偏移时间的值由系统根据VRRP不同的版本计算出来一个Skew_Time。

   VRRP通告报文时间间隔定时器，超过此时间没有收到VRRP通告报文，则认为自己是Master，并对外发送VRRP通告报文，重新进行Master的选举。

   VRRP抢占延迟定时器，Backup接收到优先级低于本地优先级的通告报文后，待一定时间——抢占延迟 时间＋Skew_Time后，才会对外发送VRRP通告报文取代原来的Master。

### VRRP主备选举

VRRP使用选举机制来确定路由器的状态，运行VRRP协议的物理路由器之间会进行选举，最终选举出一个Master，其余路由器全部都处于Backup 的状态。所有发往虚拟路由器的数据报文都由Master负责转发。如果Master 出现故障，VRRP 会从其他运行VRRP 的路由器中重新选举出一个新的Master。

VRRP中主备选举的依据主要是优先级:

- 比较优先级的大小，优先级高者当选为Master 设备.
- 当两台优先级相同的设备，如果已经存在Master，则Backup 设备不进行抢占。如果同时竞争Master，则比较接口IP 地址大小，IP 地址较大的接口所在设备当选为Master 设备。
- 其他设备作为备份设备，随时监听Master 设备的状态。

### VRRP工作原理

1. VRRP备份组中的设备根据优先级选举出Master, Maste设备通过发送免费ARP报文，将虚拟MAC地址通知给与它连接的设备或者主机,从而承担报文转发任务。
2. Master设备周期性向备份组内所有Backup设备发送VRRP通告报文，以公布其配置信息(优先级等)和工作状况。
3. 如果Master设备出现故障,VRRP备份组中的Backup设备将根据优先级选举新的Master.
4. VRRP备份组状态切换时, Master设备由一台设备切换为另外一台设备，新的Master设备会立即发送携 带虚拟路由器的虚拟MAC地址和虚拟IP地址信息的免费ARP报文,刷新与它连接的主机或设备中的MAC表项，从而把用户流量引到新的Master设备上来,整个过程对用户完全透明。
5. 原Master设备故障恢复时, 若该设备为IP地址拥有者(优先级为255 ) ,将直接切换至Master状态。若该设备优先级小于255 ,将首先切换至Backup状态，且其优先级恢复为故障前配置的优先级。
6. Backup设备的优先级高于Master设备时,由Backup设备的工作方式决定是否重新选举Master。

## 抢占模式

在VRRP 中，只有Master 才能发送VRRP 通告消息。Backup在接收到Master 发送的VRRP通告消息后，会比较自身的优先级和Master 通告消息中的优先级大小。

如果抢占模式开启，而且自己的优先级比当前Master 的优先级要高，就会将自己的状态修改为Master，并向外发送VRRP通告报文。如果抢占模式关闭，Backup 即使发现自己的优先级比当前Master 的优先级高，也不会将自己的状态修改为Master。

默认情况下，抢占模式是开启的。

> VRRP报文参考[链接](https://blog.csdn.net/yuezhilangniao/article/details/106023398)