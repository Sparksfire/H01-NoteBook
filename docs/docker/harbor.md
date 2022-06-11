## Harbor的特征

  - 多租户内容签名和验证
  - 安全性和漏洞分析
  - 审核日志记录
  - 身份集成和基于角色的访问控制
  - 实例之间的映像复制
  - 可扩展的 API 和图形用户界面


## 安装harbor

### 先决条件

在安装harbor之前需要先安装docker、docker-compose

### 下载harbor

```bash
# 解压 harbor包到 /usr/local/ 目录下
ubuntu@node1:~$ tar xf harbor-offline-installer-v2.3.5.tgz -C /usr/local/
ubuntu@node1:~$ cd /usr/local/harbor/
ubuntu@node1:/usr/local/harbor$ ls -al
total 647756
drwxr-xr-x  3 root root      4096 May 12 18:30 .
drwxr-xr-x 11 root root      4096 May 12 18:23 ..
-rw-r--r--  1 root root      3361 Apr  7 07:52 common.sh
-rw-r--r--  1 root root      5834 May 12 18:30 docker-compose.yml
-rw-r--r--  1 root root 663227387 Apr  7 07:52 harbor.v2.5.0.tar.gz
-rw-r--r--  1 root root      9917 Apr  7 07:52 harbor.yml.tmpl
-rwxr-xr-x  1 root root      2500 Apr  7 07:52 install.sh
-rw-r--r--  1 root root     11347 Apr  7 07:52 LICENSE
-rwxr-xr-x  1 root root      1881 Apr  7 07:52 prepare
ubuntu@node1:/usr/local/harbor$ 
ubuntu@node1:/usr/local/harbor$ cp harbor.yml.tmpl harbor.yml
```

### 修改配置文件

```yml
ubuntu@node1:/usr/local/harbor$ cat harbor.yml
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: harbor.example.com   # 添加主机名

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80
# 由于没有证书，可以是注释掉https
# https related config
#https:
  # https port for harbor, default is 443
  #  port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path

# # Uncomment following will enable tls communication between all harbor components
# internal_tls:
#   # set enabled to true means internal tls is enabled
#   enabled: true
#   # put your cert and key files on dir
#   dir: /etc/harbor/tls/internal

# Uncomment external_url if you want to enable external proxy
# And when it enabled the hostname will no longer used
# external_url: https://reg.mydomain.com:8433

# The initial password of Harbor admin
# It only works in first time to install harbor
# Remember Change the admin password from UI after launching Harbor.
harbor_admin_password: Harbor12345   # 修改默认登录密码

```

### 脚本安装

```bash
ubuntu@node1:/usr/local/harbor$ sudo bash install.sh 

[Step 0]: checking if docker is installed ...

Note: docker version: 20.10.16

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 2.5.0

[Step 2]: loading Harbor images ...
......安装过程省略

[Step 3]: preparing environment ...

[Step 4]: preparing harbor configs ...
prepare base dir is set to /usr/local/harbor
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir



[Step 5]: starting Harbor ...
[+] Running 10/10
 ⠿ Network harbor_harbor        Created                                                                                                                                                                                                     0.1s
 ⠿ Container harbor-log         Started                                                                                                                                                                                                     0.9s
 ⠿ Container registryctl        Started                                                                                                                                                                                                     1.9s
 ⠿ Container registry           Started                                                                                                                                                                                                     2.1s
 ⠿ Container harbor-db          Started                                                                                                                                                                                                     2.1s
 ⠿ Container harbor-portal      Started                                                                                                                                                                                                     1.8s
 ⠿ Container redis              Started                                                                                                                                                                                                     2.0s
 ⠿ Container harbor-core        Started                                                                                                                                                                                                     2.6s
 ⠿ Container nginx              Started                                                                                                                                                                                                     3.4s
 ⠿ Container harbor-jobservice  Started                                                                                                                                                                                                     3.5s
✔ ----Harbor has been installed and started successfully.----
ubuntu@node1:/usr/local/harbor$ docker ps -a
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS                             PORTS                                   NAMES
68d12fd4ef16   goharbor/harbor-jobservice:v2.5.0    "/harbor/entrypoint.…"   23 seconds ago   Up 20 seconds (health: starting)                                           harbor-jobservice
b6f361cf2421   goharbor/nginx-photon:v2.5.0         "nginx -g 'daemon of…"   23 seconds ago   Up 20 seconds (health: starting)   0.0.0.0:80->8080/tcp, :::80->8080/tcp   nginx
9384f2cdba76   goharbor/harbor-core:v2.5.0          "/harbor/entrypoint.…"   24 seconds ago   Up 21 seconds (health: starting)                                           harbor-core
fef9ad010181   goharbor/harbor-registryctl:v2.5.0   "/home/harbor/start.…"   24 seconds ago   Up 21 seconds (health: starting)                                           registryctl
dfe2dbc8f842   goharbor/harbor-db:v2.5.0            "/docker-entrypoint.…"   24 seconds ago   Up 21 seconds (health: starting)                                           harbor-db
6601d2a410e3   goharbor/registry-photon:v2.5.0      "/home/harbor/entryp…"   24 seconds ago   Up 21 seconds (health: starting)                                           registry
e74d67613439   goharbor/harbor-portal:v2.5.0        "nginx -g 'daemon of…"   24 seconds ago   Up 22 seconds (health: starting)                                           harbor-portal
7fd240846d9a   goharbor/redis-photon:v2.5.0         "redis-server /etc/r…"   24 seconds ago   Up 21 seconds (health: starting)                                           redis
a1c0f3cd333e   goharbor/harbor-log:v2.5.0           "/bin/sh -c /usr/loc…"   24 seconds ago   Up 23 seconds (health: starting)   127.0.0.1:1514->10514/tcp               harbor-log
ubuntu@node1:/usr/local/harbor$
```

安装完成之后会多两个文件，一个common，一个docker-compose.yml

```bash
ubuntu@node1:/usr/local/harbor$ ls -al
total 647756
drwxr-xr-x  3 root root      4096 May 12 18:30 .
drwxr-xr-x 11 root root      4096 May 12 18:23 ..
drwxr-xr-x  3 root root      4096 May 12 18:27 common
-rw-r--r--  1 root root      3361 Apr  7 07:52 common.sh
-rw-r--r--  1 root root      5834 May 12 18:30 docker-compose.yml
-rw-r--r--  1 root root 663227387 Apr  7 07:52 harbor.v2.5.0.tar.gz
-rw-r--r--  1 root root      9912 May 12 18:29 harbor.yml
-rw-r--r--  1 root root      9917 Apr  7 07:52 harbor.yml.tmpl
-rwxr-xr-x  1 root root      2500 Apr  7 07:52 install.sh
-rw-r--r--  1 root root     11347 Apr  7 07:52 LICENSE
-rwxr-xr-x  1 root root      1881 Apr  7 07:52 prepare
ubuntu@node1:/usr/local/harbor$ 

```

### harbor网页操作

### 浏览器访问

![](https://images.icodedream.com/images/2022/05/13/harbor-h1.png)


### 设置简体中文

![](https://images.icodedream.com/images/2022/05/13/harbor-h2.png)

### 创建用户

![](https://images.icodedream.com/images/2022/05/13/harbor-h3.png)

### 创建项目

在这里我们创建一个test的项目，在这里存储容量-1表示不限制。

![](https://images.icodedream.com/images/2022/05/13/harbor-h4.png)

接下来我们将之前创建的用户添加到该项目中。

![](https://images.icodedream.com/images/2022/05/13/harbor-h5.png)

## 本地push镜像

我们先拉取一个nginx的镜像重新打tag，进行上传测试。

```bash
ubuntu@node1:~$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
214ca5fb9032: Pull complete 
f0156b83954c: Pull complete 
5c4340f87b72: Pull complete 
9de84a6a72f5: Pull complete 
63f91b232fe3: Pull complete 
860d24db679a: Pull complete 
Digest: sha256:2c72b42c3679c1c819d46296c4e79e69b2616fa28bea92e61d358980e18c9751
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
ubuntu@node1:~$ 
```

本地服务器登录harbor

```bash
ubuntu@node1:~$ docker login -u admin -p Harbor12345 http://harbor.example.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
ubuntu@node1:~$ 
```

本地镜像上传

```bash
ubuntu@node1:~$ docker tag nginx:latest harbor.example.com/test/nginx:v1
ubuntu@node1:~$ docker images
REPOSITORY                      TAG       IMAGE ID       CREATED       SIZE
harbor.example.com/test/nginx   v1        7425d3a7c478   2 days ago    142MB
nginx                           latest    7425d3a7c478   2 days ago    142MB
ubuntu@harbor:~$ docker push harbor.example.com/test/nginx:v1
The push refers to repository [harbor.example.com/test/nginx]
feb57d363211: Pushed 
98c84706d0f7: Pushed 
4311f0ea1a86: Pushed 
6d049f642241: Pushed 
3158f7304641: Pushed 
fd95118eade9: Pushed 
v1: digest: sha256:787480bfb4297dc887f8655dbc51074ef87f16ea359baeea3af0a4dd92948124 size: 1570
ubuntu@harbor:~$ 
```

在我们创建的test仓库就可以查看到上面上传的镜像了。

![](https://images.icodedream.com/images/2022/05/13/harbor-h6.png)
![](https://images.icodedream.com/images/2022/05/13/harbor-h7.png)
![](https://images.icodedream.com/images/2022/05/13/harbor-h8.png)

## 远程push镜像

```bash
C:\Users\Administrator> docker login -u admin -p Harbor12345 http://harbor.example.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Error response from daemon: Get https://harbor.example.com/v2/: dial tcp 192.168.35.21:443: connect: connection refused
C:\Users\Administrator>
```
>[!Tip][官方说明](https://goharbor.io/docs/2.3.0/install-config/run-installer-script/#connect-http),这是因为在使用docker的仓库时，Registry为了安全性考虑，默认是需要https证书支持。除了生成证书，配置https的办法之外，我们在测试环境中，还可以通过修改docker启动文件来解决。但这里的坑太大了，官方也没有说明到底是该客户端还是服务端，改了半天服务端一点屁用都没有！大坑 。。。经过测试，最好使用域名或者hostname,一定要解析到hosts文件里面。建议生产环境还是配置https。

![](https://images.icodedream.com/images/2022/05/13/harbor-h9.png)
![](https://images.icodedream.com/images/2022/05/13/harbor-h10.png)

```bash
C:\Users\Administrator> docker login -u admin -p Harbor12345 http://harbor.example.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
C:\Users\Administrator> docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete
a9edb18cadd1: Pull complete
589b7251471a: Pull complete
186b1aaa4aa6: Pull complete
b4df32aa5a72: Pull complete
a0bcbecc962e: Pull complete
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
C:\Users\Administrator> docker tag nginx:latest harbor.example.com/test/nginx:v2
C:\Users\Administrator> docker images
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
nginx                           latest    605c77e624dd   4 months ago   141MB
harbor.example.com/test/nginx   v2        605c77e624dd   4 months ago   141MB
C:\Users\Administrator> docker push harbor.example.com/test/nginx:v2
The push refers to repository [harbor.example.com/test/nginx]
d874fd2bc83b: Pushed
32ce5f6a5106: Pushed
f1db227348d0: Pushed
b8d6e692a25e: Pushed
e379e8aedd4d: Pushed
2edcec3590a4: Pushed
v2: digest: sha256:ee89b00528ff4f02f2405e4ee221743ebc3f8e8dd0bfd5c4c20a2fa2aaa7ede3 size: 1570
C:\Users\Administrator>
```

从上面的额信息可以看出，已经push成功了，接下来去查看一下。

![](https://images.icodedream.com/images/2022/05/13/harbor-h11.png)
![](https://images.icodedream.com/images/2022/05/13/harbor-h12.png)
