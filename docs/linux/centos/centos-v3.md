
## 安装Nginx依赖

```bash
# gcc安装，nginx源码编译需要
yum -y install gcc-c++

# PCRE pcre-devel 安装，nginx 的 http 模块使用 pcre 来解析正则表达式
yum -y install  pcre pcre-devel

# zlib安装，nginx 使用zlib对http包的内容进行gzip
yum -y install zlib zlib-devel

# OpenSSL 安装，强大的安全套接字层密码库，nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http）
yum -y install  openssl openssl-devel
```

## 下载Nginx安装包

```bash
wget -c https://nginx.org/download/nginx-1.20.2.tar.gz
```

## 安装

```bash
# 根目录使用ls命令可以看到下载的nginx压缩包，然后解压
[root@localhost ~]# tar -zxvf nginx-1.20.2.tar.gz

# 创建nginx用户用户运行nginx服务
[root@localhost ~]# useradd -M -s /sbin/nologin nginx

# 解压后进入目录
[root@localhost ~]# cd nginx-1.20.2

# 使用默认配置，在这里仅为了测试，后续在完善这块
./configure

# 编译安装
[root@localhost nginx-1.20.2]# make && make install
make -f objs/Makefile
make[1]: Entering directory `/root/nginx-1.20.2'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs \
	-o objs/src/core/nginx.o \
	src/core/nginx.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs \
	-o objs/src/core/ngx_log.o \
	src/core/ngx_log.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs \
	-o objs/src/core/ngx_palloc.o \
	src/core/ngx_palloc.c
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/unix -I objs \
	-o objs/src/core/ngx_array.o \
......
......
......
	|| cp conf/scgi_params '/usr/local/conf'
cp conf/scgi_params \
	'/usr/local/conf/scgi_params.default'
test -f '/usr/local/conf/nginx.conf' \
	|| cp conf/nginx.conf '/usr/local/conf/nginx.conf'
cp conf/nginx.conf '/usr/local/conf/nginx.conf.default'
test -d '/usr/local/logs' \
	|| mkdir -p '/usr/local/logs'
test -d '/usr/local/logs' \
	|| mkdir -p '/usr/local/logs'
test -d '/usr/local/html' \
	|| cp -R html '/usr/local'
test -d '/usr/local/logs' \
	|| mkdir -p '/usr/local/logs'
make[1]: Leaving directory `/root/nginx-1.20.2'
[root@localhost nginx-1.20.2]# whereis nginx
nginx: /usr/local/nginx
[root@localhost nginx-1.20.2]# 
```

## 启动、停止nginx

```bash
[root@localhost ~]# /usr/local/nginx/sbin/nginx

[root@localhost ~]# netstat -antup | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1500/nginx: master  
[root@localhost ~]# 

/usr/local/nginx/sbin/nginx     #启动
/usr/local/nginx/sbin/nginx -s stop  #停止，直接查找nginx进程id再使用kill命令强制杀掉进程
/usr/local/nginx/sbin/nginx -s quit  #退出停止，等待nginx进程处理完任务再进行停止
/usr/local/nginx/sbin/nginx -s reload  #重新加载配置文件，修改nginx.conf后使用该命令，新配置即可生效
```

### 为方便操作，在这里把Nginx配置未系统服务

### 1.在`/usr/lib/systemd/system`目录下添加nginx.service

```bash
[root@localhost ~]# vi /usr/lib/systemd/system/nginx.service
```

```
[Unit]
Description=nginx web service
Documentation=http://nginx.org/en/docs/
After=network.target
 
[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true
 
[Install]
WantedBy=default.target
```

### 2.设置权限

```bash
[root@localhost ~]#  chmod 755 /usr/lib/systemd/system/nginx.service
```

### 3.接下来就可以使用系统命令来操作Nginx了

```bash
[root@localhost ~]# systemctl start nginx.service
[root@localhost ~]# systemctl status nginx.service
● nginx.service - nginx web service
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-04-20 01:44:16 CST; 4s ago
     Docs: http://nginx.org/en/docs/
  Process: 3235 ExecStart=/usr/local/nginx/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3233 ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 3237 (nginx)
   CGroup: /system.slice/nginx.service
           ├─3237 nginx: master process /usr/local/nginx/sbin/nginx
           └─3238 nginx: worker process

Apr 20 01:44:16 localhost.localdomain systemd[1]: Starting nginx web service...
Apr 20 01:44:16 localhost.localdomain nginx[3233]: nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
Apr 20 01:44:16 localhost.localdomain nginx[3233]: nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
Apr 20 01:44:16 localhost.localdomain systemd[1]: Started nginx web service.
[root@localhost ~]# 
```

### 4.相关操作命令

```bash
# 启动: systemctl start nginx
# 停止: systemctl stop nginx
# 重启: systemctl restart nginx
# 重新加载配置文件: systemctl reload nginx
# 查看nginx状态: systemctl status nginx
# 开机启动: systemctl enable nginx
```

结束，告辞！
