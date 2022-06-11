## Jenkins介绍

>[!Tip][Jenkins](https://www.jenkins.io/)是一款非常流行的开源持续集成的工具，广泛用于项目开发，具有自动化构建、测试和部署等功能。

## Jenkins的特征

> - 开源的Java语言开发持续集成工具，支持持续集成，持续部署。
> - 易于安装部署配置：可以通过yum安装或下载war包以及通过Docker容器等快速实现安装部署，非常方便的web界面配置管理。
> - 消息通知及测试报告：集成RSS/email，通过RSS发布构建结果或者当构建完成的时候通过email通知，生成Junit/TestNG测试报告。
> - 分布式构建：能够让多台计算机一起构建和测试。
> - 文件识别：Jenkins能够跟踪哪次构建生成那些jar，哪次构建使用那个版本的jar等。
> - 丰富的插件支持：支持扩展插件，我们甚至可以开发适合自己团队使用的工具，如git、svn、maven、docker等。

## 持续集成说明

![](https://images.icodedream.com/images/2022/05/07/Jenkins-v1.png)

- 首先，开发人员每天进行代码提交，提交到Git仓库。
- Jenkins作为持续集成工具，使用Git工具到Git仓库拉取代码到持续集成服务器，再配合JDK，Maven等软件完成代码的编译、测试、审核、打包等工作，在这个过程中每一步出错，都重新再执行一次整个流程。
- 最后，Jenkins把生成的jar包分发到测试服务器或者生产服务器，测试人员或用户就可以访问应用了。

## Gitlab简介

> - Gitlab是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的web服务。
> - Gitlab和GitHub一样属于第三方基于Git开发的产品，免费且开源（基于MIT协议），和GitHub类似，可以注册用户，任意提交自己的代码，添加ssh key等等。不同的是，Gitlab是可以部署到自己的服务器上的，数据库等一切信息都掌握在自己手里，适合团队内部协作开发。换言之，Gitlab可以看作个人版的GitHub。
> - [官方文档](https://docs.gitlab.com/)
### Gitlab的安装

!> 本次安装均为docker环境，请提前安装好docker、docker-compose

创建docker-compose.yaml
```yaml
version: '3.6'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: always
    hostname: gitlab
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://dev.gitlab.com'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - 80:80
      - 2224:22
    volumes:
      - ./srv/config:/etc/gitlab
      - ./srv/logs:/var/log/gitlab
      - ./srv/data:/var/opt/gitlab
    shm_size: '256m'
```

### 启动gitlab

```bash

[root@localhost Jenkins]# docker-compose -f docker-compose.yaml up -d
[+] Running 2/2
 ⠿ Network jenkins_default  Created                                                                                                                                                                           0.1s
 ⠿ Container gitlab         Started                                                                                                                                                                           0.6s
[root@localhost Jenkins]# docker-compose ps
NAME                COMMAND             SERVICE             STATUS               PORTS
gitlab              "/assets/wrapper"   gitlab              running (starting)   0.0.0.0:2224->22/tcp, 0.0.0.0:80->80/tcp, :::2224->22/tcp, :::80->80/tcp
[root@localhost Jenkins]# 

```

### 查看密码

```bash
[root@localhost Jenkins]# docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
Password: G6S3QL0uPrLx3i6zWhChzkRCp5+VSaIk21wXlpIaenw=
[root@localhost Jenkins]# 
```

获取到密码后，在浏览器访问`http://dev.gitlab.com`，就会看到如下图所示：![](https://images.icodedream.com/images/2022/05/07/gitlab-v1.png)

!> 注意，这里由于是测试，配置了`hosts`对于`dev.gitlab.com`的`ip`地址。

## Jenkins安装

```yaml
version: '3.6'

services:
  jenkins:
    image: jenkins/jenkins:latest
    container_name: jenkins
    hostname: jenkins
    restart: always
    ports:
      - 8099:8080
      - 50000:50000
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
```

首次启动之后，会因为没有jenkins_home目录的读写权限，会报错！需要对该目录设置权限`chmod -R a+w jenkins_home/
`

>[!TIP] 对于安装Jenkins，建议安装最新版，要不然很多插件安装都会报错！修改Jenkins安装目录下的`hudson.model.UpdateCenter.xml`文件中`<url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>`的值，改成这样就行。对于这个插件的安装真特么劝退很多人。

## 查看密码

```bash
[root@localhost ~]# docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
578e5255e8f04007ac23347f17a31011
[root@localhost ~]#  
```

<!-- 
修改插件下载地址

```bash
sed -i 's#http:\/\/updates.jekins-ci.org\/download#https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins#g' default.json && sed -i '#/http:\/\/www.google.com#https:\/\/www.baidu.com#g' default.json
```


```
ENV JENKINS_UC=https://updates.jenkins.io
ENV JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental
ENV JENKINS_INCREMENTALS_REPO_MIRROR=https://repo.jenkins-ci.org/incrementals


ENV COPY_REFERENCE_FILE_LOG=/var/jenkins_home/copy_reference_file.log
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

``` -->

未完，待续！