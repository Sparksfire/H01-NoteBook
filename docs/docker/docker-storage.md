Docker数据卷持久化的三种方式

![](https://images.icodedream.com/images/2022/05/15/types-of-mounts-volume.png)

## bind mount

可以存储在 Docker Host 文件系统的任何位置，它们甚至可能是重要的系统文件或目录，非 Docker 的进程或者 Docker 容器可能随时对其进行修改，存在潜在的安全风险。

### 语法

bind mount可以使用-v或者--volume，这个参数在单容器的情况下使用，在 swarm 集群中使用--mount，从 Docker 17.06 之后，可以统一使用参数--mount

-v or --volume 语法分为三部分，使用冒号进行分割，排列的顺序不能发生变化，每个字段所表达的含是不同的；

1. 映射到宿主机文件的目录的地址;
2. 挂载到容器上的文件目录的地址；
3. 可选字段，用来增加附加项目，比如ro，rw等等;

--mount语法由一组键值对组成，用逗号进行分割；

![](https://images.icodedream.com/images/2022/05/15/types-of-mounts-volume-1.png)

两者的区别：

1. 在使用-v的时候，如果Docker Host不存在要挂载的文件或者目录，Docker将会自动进行创建，通常是一个目录。
2. 在使用-mount的时候，如果Docker Host不存在要挂载的文件或者目录，Docker不会自动创建目录，并生成一个错误。

### -v 的使用

1. 启动一个nginx实例，--name指定为nginx1，将容器内端口映射到宿主机8001，将宿主机~/nginx1/app/docker映射到容器内/usr/share/nginx/html/目录；

```bash
ubuntu@ubuntu:~$ ls
get-docker.sh
ubuntu@ubuntu:~$ docker run -d --name nginx1 -v ~/nginx1/app/docker:/usr/share/nginx/html/ nginx:latest
77b7d7927d2e11dfa29b024f262660185b81ecee331307fa7fbc4e6d5a815f4f
ubuntu@ubuntu:~$ ls
get-docker.sh  nginx1
ubuntu@ubuntu:~$ 
```
2. 查看挂载目录

```bash
ubuntu@ubuntu:~$ docker inspect nginx1 |  grep Mounts -A 9
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/ubuntu/nginx1/app/docker",
                "Destination": "/usr/share/nginx/html",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
ubuntu@ubuntu:~$ 
```
### -mount 的使用

1. 启动一个nginx实例，--name指定为nginx2，将容器内端口映射到宿主机8002，将宿主机~/nginx2/app/docker映射到容器内/usr/share/nginx/html/目录；

```bash
ubuntu@ubuntu:~$ docker run -d --name nginx2 -p 8002:80 --mount type=bind,source=~/nginx2/app/docker,target=/usr/share/nginx/html/ nginx:latest
docker: Error response from daemon: invalid mount config for type "bind": invalid mount path: '~/nginx2/app/docker' mount path must be absolute.
See 'docker run --help'.
ubuntu@ubuntu:~$ 
```

可以看出上面报错了，意思是说我们需要把source的路径指定为绝对路径。

```bash
ubuntu@ubuntu:~$ docker run -d --name nginx2 -p 8002:80 --mount type=bind,source=/home/ubuntu/nginx2/app/docker,target=/usr/share/nginx/html/ nginx:latest
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /home/ubuntu/nginx2/app/docker.
See 'docker run --help'.
ubuntu@ubuntu:~$ 
```

可以看出，又报错了！为啥呢，上面其实已经说过了，在使用-mount的时候，如果Docker Host不存在要挂载的文件或者目录，Docker不会自动创建目录，并生成一个错误。

```bash
ubuntu@ubuntu:~$ mkdir -p nginx2/app/docker 
ubuntu@ubuntu:~$ docker run -d --name nginx2 -p 8002:80 --mount type=bind,source=/home/ubuntu/nginx2/app/docker,target=/usr/share/nginx/html/ nginx:latest
29f82125ed6f1c90655fa7007abbda5584fbc9708af37a61f3978986b7679e11
ubuntu@ubuntu:~$ 
```

### 只读

启动一个nginx实例，--name指定为nginx3，将容器内端口映射到宿主机8003，将将宿主机~/nginx3/app/docker映射到容器内/usr/share/nginx/html/目录，并且设置为只读；

```bash
ubuntu@ubuntu:~$ docker run -d --name nginx3 -p 8003:80 -v ~/nginx3/app/docker:/usr/share/nginx/html/:ro nginx:latest
c11402167fde16de05edbee5cf2ba2dec2dc2db2157e3f041681a59a79501360
ubuntu@ubuntu:~$ 
```

接下来我们试着在容器内部写入一个文件，看看会不会和同步到宿主机目录。

```bash
ubuntu@ubuntu:~$ docker exec -it nginx3 /bin/bash
root@c11402167fde:/# cd /usr/share/nginx/html/
root@c11402167fde:/usr/share/nginx/html# touch test.txt
touch: cannot touch 'test.txt': Read-only file system
root@c11402167fde:/usr/share/nginx/html# 
```

可以从上述信息中看出，报错了！那么我们在宿主机目录创建试试。

```bash
# 宿主机
ubuntu@ubuntu:~/nginx3/app/docker$ sudo touch test.txt
[sudo] password for ubuntu: 
ubuntu@ubuntu:~/nginx3/app/docker$ ls
test.txt
ubuntu@ubuntu:~/nginx3/app/docker$ 

---------------------------------------------------------------------------------
# 容器
root@c11402167fde:/usr/share/nginx/html# ls
test.txt
root@c11402167fde:/usr/share/nginx/html# 
```
可以看出，只有宿主机可以进行数据修改。我们来查看一下挂载情况。

```bash
ubuntu@ubuntu:~$ docker inspect nginx3 | grep Mounts -A 9
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/ubuntu/nginx3/app/docker",
                "Destination": "/usr/share/nginx/html",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            }
        ],
ubuntu@ubuntu:~$ 
```

readonly设置了只读权限，在容器中是无法对bind mount数据进行修改的。

---

## volume

存储在 Docker Host 文件系统的一个路径下，这个路径是由 Docker 来进行管理，路径默认是 /var/lib/docker/volumes/，非 Docker 的进程不能去修改这个路径下面的文件，所以说volumes是持久存储数据最好的一种方式。

### 语法

-v与bind mount语法类似，在不指定主机地址的时候，就是采用volume的方式进行挂载，分为三部分:

1. 指定卷的名称；
2. 容器指定的挂载卷的地址；
3. 可选字段，用来增加附加项目，比如ro，rw等等;

--mount与bind mount语法类似，使用逗号分隔:

![](https://images.icodedream.com/images/2022/05/16/types-of-mounts-volume-2.png)

### 使用方法

1. 创建名为test的卷：

```bash
ubuntu@ubuntu:~$ docker volume create test
test
ubuntu@ubuntu:~$ docker volume ls
DRIVER    VOLUME NAME
local     test
root@ubuntu:~# ls /var/lib/docker/volumes/
backingFsBlockDev  metadata.db  test
root@ubuntu:~# 
```

2. 启动一个nginx实例，--name指定为nginx4，将容器内端口映射到宿主机8004，将宿主机~/nginx4/app/docker映射到容器内/usr/share/nginx/html/目录；

```bash
# --mount
docker run -d -p 8004:80  --name nginx4 --mount source=test,target=/usr/share/nginx/html nginx:latest
# -v命令
docker run -d -p 8004:80 --name nginx4 -v test:/usr/share/nginx/html nginx:latest
---------------------------------------------------------------------------------

ubuntu@ubuntu:~$ docker run -d -p 8004:80 --name nginx4 -v test:/usr/share/nginx/html nginx:latest
38d201b6e2126e9ef559f9ad65c49995d163b61ae692dbaedaea62939aee8678
ubuntu@ubuntu:~$ 
```

3. 查看容器的挂载

```bash
ubuntu@ubuntu:~$ docker inspect nginx4 | grep Mounts -A 11
        "Mounts": [
            {
                "Type": "volume",
                "Name": "test",
                "Source": "/var/lib/docker/volumes/test/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
ubuntu@ubuntu:~$ 
```

4. 访问localhost:8004

```bash
ubuntu@ubuntu:~$ curl localhost:8004
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ubuntu@ubuntu:~$ 
```

5. 查看一下test卷_data目录

```bash
root@ubuntu:~# ls /var/lib/docker/volumes/test/_data/
50x.html  index.html
root@ubuntu:~# 
```
6. 在volume中创建test.txt，然后进去容器目录查看。

```bash
root@ubuntu:~# touch /var/lib/docker/volumes/test/_data/test.txt
root@ubuntu:~# ls /var/lib/docker/volumes/test/_data/
50x.html  index.html  test.txt
root@ubuntu:~# 

--------------------------------------------------------

root@ubuntu:~# docker exec -it nginx4 ls /usr/share/nginx/html
50x.html  index.html  test.txt
root@ubuntu:~# 

# 或进入容器查看

root@ubuntu:~# docker exec -it nginx4 /bin/bash
root@38d201b6e212:/# ls /usr/share/nginx/html/
50x.html  index.html  test.txt
root@38d201b6e212:/# 
```
---

## tmpfs mount

只存储在 Docker Host 的系统内存中，不会写入到系统的文件系统中，不会持久存储。

### 语法

在单容器的情况下使用--tmpfs，并且不能指定参数，在集群的情况下使用--mount，可以指定一些参数；

![](https://images.icodedream.com/images/2022/05/16/types-of-mounts-volume-3.png)

### 使用方法

```bash
docker run -d -p 8005:80 --name nginx5 --mount type=tmpfs,target=/usr/share/nginx/html nginx:latest
```

## 共享容器数据卷

1. 启动一个nginx实例，--name指定为nginx6，将容器内端口映射到宿主机8006，将宿主机~/nginx6/app/docker映射到容器内/usr/share/nginx/html/目录，进入容器后创建一个test.txt的测试文件。

2. 再启动一个nginx实例，--name指定为nginx7，将容器内端口映射到宿主机8007，且共享nginx6的数据；

```bash
ubuntu@ubuntu:~$ docker run -d --name nginx6 -p 8006:80 -v ~/nginx6/app/docker:/usr/share/nginx/html/ nginx:latest
d8b337f027172c9affa521316f2cbd79596bc55cebc81d03949f855c9eeb3d6e
ubuntu@ubuntu:~$ docker exec -it nginx6 touch /usr/share/nginx/html/test.txt
ubuntu@ubuntu:~$ docker exec -it nginx6 ls /usr/share/nginx/html
test.txt
ubuntu@ubuntu:~$ ls nginx6/app/docker/
test.txt
ubuntu@ubuntu:~$ docker run -d --name nginx7 -p 8007:80 --volumes-from nginx6 nginx:latest
9b1e6ddf9b875022fcfc1945f85d12a9bf780295e5c933fc1457953a280d1db1
ubuntu@ubuntu:~$ docker exec -it nginx7 ls /usr/share/nginx/html
test.txt
ubuntu@ubuntu:~$ 
```

















## UnionFS

Linux的命名空间和控制组分别解决了不同资源隔离的问题，前者解决了进程、网络已经文件系统的隔离，后者实现了CPU、内存等资源的隔离，但是在Docker中海油另一个非常重要的问题需要解决，也就是镜像。

镜像到底什么呢，它又是如何组成和组织的呢？其实Docker镜像本质就是一个压缩包，饿哦们可以使用命令讲一个Docker镜像中的文件导出：

```bash
ubuntu@ubuntu:~$ docker export $(docker create busybox) | tar -C rootfs -xvf -
ubuntu@ubuntu:~$ ls rootfs/
bin  dev  etc  home  proc  root  sys  tmp  usr  var
ubuntu@ubuntu:~$ 
```

从上面的信息可以看出，当前我们这个镜像busybox镜像中的目录结构与Linux操作系统的根目录中的内容没有太多的区别，那么可以说Docker镜像就是一个文件。

## 存储驱动

如果内核中支持多个存储驱动程序，则在未明确配置存储驱动程序的情况下，Docker会优先列出要使用的存储驱动程序，前提是该存储驱动程序满足先决条件。

我们可以从Docker源码中可以看出，Docker支持的存储驱动有以下那么多！

```go
var (
	// List of drivers that should be used in an order
	priority = "btrfs,zfs,overlay2,fuse-overlayfs,aufs,overlay,devicemapper,vfs"

	// FsNames maps filesystem id to name of the filesystem.
	FsNames = map[FsMagic]string{
		FsMagicAufs:        "aufs",
		FsMagicBtrfs:       "btrfs",
		FsMagicCramfs:      "cramfs",
		FsMagicEcryptfs:    "ecryptfs",
		FsMagicExtfs:       "extfs",
		FsMagicF2fs:        "f2fs",
		FsMagicFUSE:        "fuse",
		FsMagicGPFS:        "gpfs",
		FsMagicJffs2Fs:     "jffs2",
		FsMagicJfs:         "jfs",
		FsMagicNfsFs:       "nfs",
		FsMagicOverlay:     "overlayfs",
		FsMagicRAMFs:       "ramfs",
		FsMagicReiserFs:    "reiserfs",
		FsMagicSmbFs:       "smb",
		FsMagicSquashFs:    "squashfs",
		FsMagicTmpFs:       "tmpfs",
		FsMagicUnsupported: "unsupported",
		FsMagicVxFS:        "vxfs",
		FsMagicXfs:         "xfs",
		FsMagicZfs:         "zfs",
	}
)
```