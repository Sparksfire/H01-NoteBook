## 镜像(Image)
Docker镜像是一个特殊的文件系统，包含了程序运行时候所需要的资源和环境。镜像不包含任何动态数据，其内容在构建之后也不会被改变。镜像就是模板，可以用来创建Docker容器，另外Docker提供了很简单的机制来创建镜像和更新现有的镜像，用户还可以直接从镜像仓库下载已经做好的镜像来直接使用。

## 容器(Container)
容器就是运行镜像的，镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体，容器可以被创建、启动、停止、删除、暂停等。每个容器都是互相隔离的，保证安全的平台，容器可以理解为简易版的Linux环境（包括root用户权限、镜像空间、用户空间和网络空间等）和运行再其中的应用程序。

## 仓库(Repository)
仓库就是存放镜像的地方，仓库中又包含了多个镜像，每个镜像有不同的标签，用来区分不同的镜像版本，仓库分为两种，公有和私有仓库，最大的公开仓库是Docker Hub，存放了数量庞大的镜像供用户下载，这里仓库的概念与Git类似，Registry可以理解为Github这样的托管服务。


![](https://images.icodedream.com/images/2022/05/12/docker-v1.png)

## Docker运行方式

Docker是一个Client-Server结构，Docker守护进程运行在主机上，客户端与Dcoker通过Socket访问，守护进程接收客户端的命令并且管理运行主机的容器，容器是一个运行环境，就是我们的集装箱；

![](https://images.icodedream.com/images/2022/05/12/docker-v4.png)


## Docker基本命令

### 基础命令

```bash
# 获取docker系统配置的信息
[root@VM-20-13-centos ~]# docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.8.1-docker)
  scan: Docker Scan (Docker Inc., v0.17.0)

Server:
 Containers: 4
  Running: 4
  Paused: 0
  Stopped: 0
 Images: 13
 Server Version: 20.10.14
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc io.containerd.runc.v2 io.containerd.runtime.v1.linux
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc version: v1.0.3-0-gf46b6ba
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-1160.45.1.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.7GiB
 Name: VM-20-13-centos
 ID: 73ET:PNXG:U6PL:CAPJ:HYP2:7KSU:DIWC:O2HA:MLMA:E6DZ:GQ5J:UMQ5
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://5ev99jhr.mirror.aliyuncs.com/
 Live Restore Enabled: false

[root@VM-20-13-centos ~]# 

# 获取docker版本
[root@VM-20-13-centos ~]# docker version
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
[root@VM-20-13-centos ~]# 

# 获取docker帮助文档
[root@VM-20-13-centos ~]# docker help

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  app*        Docker App (Docker Inc., v0.9.1-beta3)
  builder     Manage builds
  buildx*     Docker Buildx (Docker Inc., v0.8.1-docker)
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  scan*       Docker Scan (Docker Inc., v0.17.0)
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

To get more help with docker, check out our guides at https://docs.docker.com/go/guides/
[root@VM-20-13-centos ~]# 
```

### 镜像命令

```bash
# 查看镜像相关的信息:
# -a 查看所有镜像;
# -q 查看容器id;

[root@VM-20-13-centos ~]# docker images
REPOSITORY                                            TAG                   IMAGE ID       CREATED         SIZE
docsify/v1                                            latest                708a6abad9be   2 weeks ago     1.02GB
mondedie/flarum                                       latest                f42b5ab11132   2 months ago    139MB
klakegg/hugo                                          latest                ae73c881d883   4 months ago    50.6MB
wordpress                                             latest                c3c92cc3dcb1   4 months ago    616MB
node                                                  latest                a283f62cb84b   4 months ago    993MB
nmtan/chevereto                                       latest                0828869c7511   5 months ago    539MB
registry.cn-hangzhou.aliyuncs.com/mindoc-org/mindoc   v2.1-beta.5           0dab9bf83181   6 months ago    922MB
mondedie/flarum                                       stable                64297f13d28e   11 months ago   131MB
wordpress                                             4.8.3-php7.0-apache   ef806f94f4c2   4 years ago     421MB
[root@VM-20-13-centos ~]# docker images -a
REPOSITORY                                            TAG                   IMAGE ID       CREATED         SIZE
<none>                                                <none>                fac8f8fde8f6   2 weeks ago     1.02GB
docsify/v1                                            latest                708a6abad9be   2 weeks ago     1.02GB
<none>                                                <none>                dd378fd0ea25   2 weeks ago     1.02GB
<none>                                                <none>                137120ec45f0   2 weeks ago     993MB
<none>                                                <none>                56329b0ee78c   2 weeks ago     993MB
mondedie/flarum                                       latest                f42b5ab11132   2 months ago    139MB
klakegg/hugo                                          latest                ae73c881d883   4 months ago    50.6MB
wordpress                                             latest                c3c92cc3dcb1   4 months ago    616MB
node                                                  latest                a283f62cb84b   4 months ago    993MB
nmtan/chevereto                                       latest                0828869c7511   5 months ago    539MB
registry.cn-hangzhou.aliyuncs.com/mindoc-org/mindoc   v2.1-beta.5           0dab9bf83181   6 months ago    922MB
mondedie/flarum                                       stable                64297f13d28e   11 months ago   131MB
wordpress                                             4.8.3-php7.0-apache   ef806f94f4c2   4 years ago     421MB
[root@VM-20-13-centos ~]# docker images -q
708a6abad9be
f42b5ab11132
ae73c881d883
c3c92cc3dcb1
a283f62cb84b
0828869c7511
0dab9bf83181
64297f13d28e
ef806f94f4c2
[root@VM-20-13-centos ~]# 

docker search # 搜索容器信息
-filter=STARS=3000 # 关注度大于300以上的;
docker pull # 下载镜像，采用分层下载，采用联合文件系统，默认是新的版本的
dcoker pull # mysql:5.7  dcoker pull 镜像名称:版本号(Tag);
docker rmi  # 删除镜像
docker rmi -f 镜像ID
docker rmi -f $(docker iamges -aq) # 删除全部的镜像id
```

### 容器命令
docker run 是Docker中最为核心的一个命令，用于新建并启动容器
```bash
-name="名称"  容器名称;
-d  使用后台交互的方式;
-it  使用交互方式，进入到容器内部;
-p  用于将容器的端口暴露给宿主机的端口，格式为：hostPort:containerPort ，通过端口的暴露，可以让外部主机能够访问容器内的应用;
-P  随机指定端口;
-c 用于给运行在容器中的所有进程分配 CPU 的 shares 值，这是一个相对权重，实际的处理速度与宿主机的 CPU 相关
-m 用于限制为容器中所有进程分配的内存总量，以 B、K、M、G 为单位；
docker ps 列出正在运行的容器
-a 列出当前正在运行的容器和历史运行过的容器;
-n=? 显示最近创建的容器;
-q 只显示容器的编号;
```

### 退出容器

```bash
exit 容器停止并退出;
Ctrl + P + Q 容器停止并退出;
```

### 删除容器
```bash
docker rm containerid删除指定的容器，不能删除正在运行的容器;
docker ps -a -q | xargs docker rm 删除所有的容器;
```

### 启动和停止容器的操作

```bash
docker start containerid         启动容器;
docker restart containerid      重启容器;
docker stop containerid         停止容器;
docker kill containerid           强杀容器;
docker logs 查看docker logs的日志
--details 显示日志详情；
-f  跟随日志输出显示；
--tail 从末尾开始显示指定行的数据；
-t 显示时间戳；
--since 开始时间；
--until 结束时间；
docker top 查看容器的进程信息
docker inspect 查看镜像的元数据
```

### 进入正在运行的容器

```bash
docker exec -it containerid
docker attach containerid
```
### 容器拷贝文件到主机上

```bash
docker cp containerid 容器内路径 主机路径；
```