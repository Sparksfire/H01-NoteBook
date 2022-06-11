
Yum是CentOS默认包管理器，yum主要功能是更方便的添加/删除/更新RPM包，自动解决包的依赖性问题，便于管理大量系统的更新问题，其理念是使用一个中心仓库(repository)管理一部分甚至一个distribution的应用程序相互关系，根据计算出来的软件依赖关系进行相关的升级、安装、删除等操作，减少了Linux用户一直头痛的dependencies的问题。可以同时配置多个资源库(Repository)，简洁的配置文件(/etc/yum.conf)，自动解决增加或删除rpm包时遇到的依赖性问题，保持与RPM数据库的一致性。
## 备份系统原Base源，出现意外情况可以随时切换。

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup 
```

## 根据当前centos版本下载对应的源。

```bash
#在这里以centos 7版本为例 
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

如果在这里出现wget命令不生效，那么就说明当前系统没有安装wget工具，那么可以使用命令`yum -y install wget`进行安装，或者使用curl命令下载

```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

## 添加epel源

> EPEL (Extra Packages for Enterprise Linux，企业版Linux的额外软件包) 是Fedora小组维护的一个软件仓库项目，为RHEL/CentOS提供他们默认不提供的软件包。这个源兼容RHEL及像CentOS和Scientific Linux这样的衍生版本。
>
> 我们可以很容易地通过yum命令从EPEL源上获取很多在CentOS自带源上没有的软件。EPEL提供的软件包大多基于其对应的Fedora软件包，不会与企业版Linux发行版本的软件发生冲突或替换其文件。
>
> RHEL/CentOS系统有许多第三方源，比较流行的比如RpmForge，RpmFusion，EPEL，Remi等等。然而需要引起注意的是，如果系统添加了多个第三方源，可能会因此产生冲突——一个软件包可以从多个源获取，一些源会替换系统的基础软件包，从而可能会产生意想不到的错误。已知的就有Rpmforge与EPEL会产生冲突。对于这些问题我们建议，调整源的优先权或者有选择性的安装源，但是这需要复杂的操作，如果你不确定如何操作，推荐你只安装一个第三方源。

```bash
# 注意备份
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

## 清理缓存并生成新的缓存

使用命令清理缓存并生成新的缓存，使下载的源生效

```bash
yum clean all
yum makecache
yum -y update
```
## 常用的yum命令

```bash
yum install softwarename  #安装
yum remove softwarename #卸载软件
yum list softwarename #查看软件源中是否有此软件
yum list all #列出所有软件名称
yum list installed #列出已经安装的软件名称
yum list available #列出可以用yum安装的软件
yum clean all #清空yum缓存
yum search softwareinfo #根据软件信息搜索软件名字（如，使用search web搜索web浏览器）
yum whatprovides filename #在yum源中查找包含filename文件的软件包（如，whatprovides rm搜索汉含rm的软件，命令实质上是文件）
yum update #更新软件，会存在未知问题，一般不对服务器升降级
yum history #查看系统软件改变历史
yum reinstall softwarename #重新安装
yum info softwarename #查看软件信息
yum groups list #查看软件组信息
yum groups info softwarename #查看软件组内包含的软件
yum groups install softwarename #安装组件
yum groups remove softwarename #卸载组件
yum clean all #清理缓存
```