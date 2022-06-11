![logo](https://letsencrypt.org/images/letsencrypt-logo-horizontal.svg)


>acme.sh 是一款方便,强大的 Let's Encrypt 域名证书申请续签程序.支持一键脚本和 docker 部署.支持 http 和 DNS 两种域名验证方式,其中包括手动,自动 DNS 及 DNS alias 模式方便各种环境和需求.可同时申请合并多张单域名,泛域名证书,并自动续签证书和部署到项目。

>![logo](https://github.githubassets.com/favicons/favicon.png ':size=32x32')
<a  href ="https://github.com/acmesh-official/acme.sh">Github</a>
<a  href ="https://letsencrypt.org/zh-cn/docs/">官方文档</a>

>[!TIP]本文以DNS API验证模式并自动部署到Nginx环境为例,详细参考上述官方文档。

获取DNS API

通过域名服务商提供的API秘钥，让acme.sh自动创建域名验证记录申请域名证书。acme.sh支持非常多的域名服务商，详细可查看<a  href ="https://github.com/acmesh-official/acme.sh/wiki/dnsapi">官方dnsapi</a>，在这里以腾讯云DNSPOD为例。

腾讯云

腾讯云域名管理由DNSPOD管理，接下里我们到DNSPOD管理控制台，创建秘钥。
![](../_images/_linux/dnspod-1.png)

!>为了安全期间，在这里已开启IP 白名单

![](../_images/_linux/dnspod-2.png)

按照官方文档说明，设置API格式为：
```bash
# 修改为自己的
export DP_Id="123456"
export DP_Key="078dxxxxxxxxxxxxxxxx45x746xxxxx7"
```

安装acme.sh
```bash
[root@VM-0-14-centos ~]# curl https://get.acme.sh | sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   937    0   937    0     0    824      0 --:--:--  0:00:01 --:--:--   824
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  210k  100  210k    0     0   152k      0  0:00:01  0:00:01 --:--:--  152k
[Thu Apr 28 19:47:20 CST 2022] Installing from online archive.
[Thu Apr 28 19:47:20 CST 2022] Downloading https://github.com/acmesh-official/acme.sh/archive/master.tar.gz
[Thu Apr 28 19:47:22 CST 2022] Extracting master.tar.gz
[Thu Apr 28 19:47:22 CST 2022] It is recommended to install socat first.
[Thu Apr 28 19:47:22 CST 2022] We use socat for standalone server if you use standalone mode.
[Thu Apr 28 19:47:22 CST 2022] If you don't use standalone mode, just ignore this warning.
[Thu Apr 28 19:47:22 CST 2022] Installing to /root/.acme.sh
[Thu Apr 28 19:47:22 CST 2022] Installed to /root/.acme.sh/acme.sh
[Thu Apr 28 19:47:22 CST 2022] Installing alias to '/root/.bashrc'
[Thu Apr 28 19:47:22 CST 2022] OK, Close and reopen your terminal to start using acme.sh
[Thu Apr 28 19:47:22 CST 2022] Installing alias to '/root/.cshrc'
[Thu Apr 28 19:47:22 CST 2022] Installing alias to '/root/.tcshrc'
[Thu Apr 28 19:47:22 CST 2022] Installing cron job
[Thu Apr 28 19:47:22 CST 2022] Good, bash is found, so change the shebang to use bash as preferred.
[Thu Apr 28 19:47:23 CST 2022] OK
[Thu Apr 28 19:47:23 CST 2022] Install success!
[root@VM-0-14-centos ~]# source ~/.bashrc
[root@VM-0-14-centos ~]# acme.sh -v
https://github.com/acmesh-official/acme.sh
v3.0.3
[root@VM-0-14-centos ~]# 
```
安装成功后执行 `source ~/.bashrc` 以确保脚本所设置的别名生效。那么接下来我们就可以使用`acme.sh`命令了。如图所示，acme版本号是v3.0.3

申请签发SSL证书
- 配置DNS API
  
  编辑`acme.sh`安装目录下的`account.conf`,把前面设置的DNS API写入到该文件中。
  ```shell
  [root@VM-0-14-centos ~]# ls -al
    total 100
    dr-xr-x---.  9 root root    4096 Apr 28 19:47 .
    dr-xr-xr-x. 19 root root    4096 Apr 28 19:56 ..
    drwx------   5 root root    4096 Apr 28 19:47 .acme.sh
    -rw-------   1 root root    5632 Apr 28 19:56 .bash_history
    -rw-r--r--.  1 root root      18 Dec 29  2013 .bash_logout
    -rw-r--r--.  1 root root     176 Dec 29  2013 .bash_profile
    -rw-r--r--.  1 root root     207 Apr 28 19:47 .bashrc
    drwxr-xr-x   4 root root    4096 Nov 23 19:02 .cache
    drwxr-xr-x   3 root root    4096 Mar  7  2019 .config
    -rw-r--r--.  1 root root     136 Apr 28 19:47 .cshrc
    -rw-r--r--   1 root root   18617 Mar 24 23:25 get-docker.sh
    drwxr-xr-x   2 root root    4096 Mar 24 23:18 .pip
    drwxr-----   3 root root    4096 Mar 24 23:26 .pki
    -rw-r--r--   1 root root      73 Mar 24 23:18 .pydistutils.cfg
    drwx------   2 root root    4096 Apr 28 00:36 .ssh
    -rw-------   1 root docker 12288 Mar 24 23:40 .sudo.swp
    -rw-r--r--.  1 root root     165 Apr 28 19:47 .tcshrc
    -rw-------   1 root root       0 Mar 24 23:18 .viminfo
    drwxr-xr-x   5 root root    4096 Apr 28 00:37 .vscode-server
    [root@VM-0-14-centos ~]#  vi .acme.sh/account.conf
  ```
  ![](../_images/_linux/dnspod-3.png)
  
  !> 需要配置域名服务商可以参考相关:<a href="https://github.com/acmesh-official/acme.sh/issues/2055">issues</a>

- 申请证书
  
  !>关于ECC证书，我们只需要在末尾添加`--keylength ec-256`:
  ```bash
  acme.sh --issue --dns dns_dp -d example.com -d www.example.com --keylength ec-256
  ```
  可选长度如下：
  - ec-256 (prime256v1, “ECDSA P-256”)
  - ec-384 (secp384r1, “ECDSA P-384”)
  - ec-521 (secp521r1, “ECDSA P-521”, which is not supported by Let’s Encrypt yet.)
  
  单域名
  ```bash
  acme.sh --issue --dns dns_dp -d example.com -d www.example.com
  ```
  多域名
  ```bash
  acme.sh --issue -d example.com -d www.example.com -d cp.example.com 
  ```
  在这里`-w`指定的是web服务器的根目录，`example.com`就是你要签发证书的域名。颁发好的证书存放在`~/.acme.sh/example.com/`。

  以下示例以`icodedream.com`为例，申请泛域名证书，执行以下命令.

  !>注意：acme.sh 2.x版本默认使用Let’s Encrypt作为服务提供商，3.x之后默认使用的是ZeroSSL作为证书颁发机构。在颁发新证书之前需要进行账户注册（一次性）。点击查看<a href="https://github.com/acmesh-official/acme.sh/wiki/ZeroSSL.com-CA">文档</a>

  - 使用电子邮件注册（将下面example@email.com改成你自己的邮箱）
  
  ```bash
  [root@VM-0-14-centos ~]# acme.sh --register-account -m example@email.com --server zerossl
  [Thu Apr 28 21:14:57 CST 2022] No EAB credentials found for ZeroSSL, let's get one
  [Thu Apr 28 21:15:00 CST 2022] Registering account: https://acme.zerossl.com/v2/DV90
  [Thu Apr 28 21:15:17 CST 2022] Registered
  [Thu Apr 28 21:15:17 CST 2022] ACCOUNT_THUMBPRINT='M4RxDXbdHgv_5L-SLsqJ7CdgsqBNpJWPKt3Nv1erBMC'
  [root@VM-0-14-centos ~]# 
  ```

  - 设置默认服务商为ZeroSSL
  
  ```bash
  [root@VM-0-14-centos ~]# acme.sh --set-default-ca  --server zerossl
  [Thu Apr 28 21:20:57 CST 2022] Changed default CA to: https://acme.zerossl.com/v2/DV90
  [root@VM-0-14-centos ~]# 
  ```
  
  - 执行以下命令申请泛域名证书
  
  ```bash
  [root@VM-0-14-centos ~]# acme.sh --issue --dns dns_dp -d icodedream.com -d *.icodedream.com --keylength ec-256
  [Thu Apr 28 21:29:24 CST 2022] Using CA: https://acme.zerossl.com/v2/DV90
  [Thu Apr 28 21:29:24 CST 2022] Creating domain key
  [Thu Apr 28 21:29:25 CST 2022] The domain key is here: /root/.acme.sh/icodedream.com_ecc/icodedream.com.key
  [Thu Apr 28 21:29:25 CST 2022] Multi domain='DNS:icodedream.com,DNS:*.icodedream.com'
  [Thu Apr 28 21:29:25 CST 2022] Getting domain auth token for each domain
  [Thu Apr 28 21:29:47 CST 2022] Getting webroot for domain='icodedream.com'
  [Thu Apr 28 21:29:47 CST 2022] Getting webroot for domain='*.icodedream.com'
  [Thu Apr 28 21:29:47 CST 2022] Adding txt value: l6gFicm0MzjAoGFubCG5QuDhmodAOYUllcUk1rdo4rw for domain:  _acme-challenge.icodedream.com
  [Thu Apr 28 21:29:47 CST 2022] Adding record
  [Thu Apr 28 21:29:48 CST 2022] The txt record is added: Success.
  [Thu Apr 28 21:29:48 CST 2022] Adding txt value: 1sazX4RrUnXH4apATdAxNj_jrD6yN9ogUWsjrj7ZqBI for domain:  _acme-challenge.icodedream.com
  [Thu Apr 28 21:29:48 CST 2022] Adding record
  [Thu Apr 28 21:29:48 CST 2022] The txt record is added: Success.
  [Thu Apr 28 21:29:48 CST 2022] Let's check each DNS record now. Sleep 20 seconds first.
  [Thu Apr 28 21:30:09 CST 2022] You can use '--dnssleep' to disable public dns checks.
  [Thu Apr 28 21:30:09 CST 2022] See: https://github.com/acmesh-official/acme.sh/wiki/dnscheck
  [Thu Apr 28 21:30:09 CST 2022] Checking icodedream.com for _acme-challenge.icodedream.com
  [Thu Apr 28 21:30:12 CST 2022] Domain icodedream.com '_acme-challenge.icodedream.com' success.
  [Thu Apr 28 21:30:12 CST 2022] Checking icodedream.com for _acme-challenge.icodedream.com
  [Thu Apr 28 21:30:14 CST 2022] Domain icodedream.com '_acme-challenge.icodedream.com' success.
  [Thu Apr 28 21:30:14 CST 2022] All success, let's return
  [Thu Apr 28 21:30:14 CST 2022] Verifying: icodedream.com
  [Thu Apr 28 21:30:21 CST 2022] Processing, The CA is processing your order, please just wait. (1/30)
  [Thu Apr 28 21:30:30 CST 2022] Success
  [Thu Apr 28 21:30:30 CST 2022] Verifying: *.icodedream.com
  [Thu Apr 28 21:30:43 CST 2022] Processing, The CA is processing your order, please just wait. (1/30)
  [Thu Apr 28 21:30:48 CST 2022] Success
  [Thu Apr 28 21:30:48 CST 2022] Removing DNS records.
  [Thu Apr 28 21:30:48 CST 2022] Removing txt: l6gFicm0MzjAoGFubCG5QuDhmodAOYUllcUk1rdo4rw for domain: _acme-challenge.icodedream.com
  [Thu Apr 28 21:30:49 CST 2022] Removed: Success
  [Thu Apr 28 21:30:49 CST 2022] Removing txt: 1sazX4RrUnXH4apATdAxNj_jrD6yN9ogUWsjrj7ZqBI for domain: _acme-challenge.icodedream.com
  [Thu Apr 28 21:30:50 CST 2022] Removed: Success
  [Thu Apr 28 21:30:50 CST 2022] Verify finished, start to sign.
  [Thu Apr 28 21:30:50 CST 2022] Lets finalize the order.
  [Thu Apr 28 21:30:50 CST 2022] Le_OrderFinalize='https://acme.zerossl.com/v2/DV90/order/DYF3dJZOaTxSYBi6RMR_pg/finalize'
  [Thu Apr 28 21:30:58 CST 2022] Order status is processing, lets sleep and retry.
  [Thu Apr 28 21:30:58 CST 2022] Retry after: 15
  [Thu Apr 28 21:31:14 CST 2022] Polling order status: https://acme.zerossl.com/v2/DV90/order/DYF3dJZOaTxSYBi6RMR_pg
  [Thu Apr 28 21:31:23 CST 2022] Downloading cert.
  [Thu Apr 28 21:31:23 CST 2022] Le_LinkCert='https://acme.zerossl.com/v2/DV90/cert/wgIyNNSz_B5vKgxxFhpqwQ'
  [Thu Apr 28 21:31:30 CST 2022] Cert success.
  --keylength ec-256
  ```

  生成过程中，几分钟左右即可完成，在验证的过程中会自动解析一个TXT记录，校验完成后会自动删除，所以建议大家选择DNS校验，这里就不截图了，感兴趣的可以是dnspod域名继续的记录查看。待验证通过后即表示申请成功，如下所示：
  ```bash
  [Thu Apr 28 21:31:30 CST 2022] Your cert is in: /root/.acme.sh/icodedream.com_ecc/icodedream.com.cer
  [Thu Apr 28 21:31:30 CST 2022] Your cert key is in: /root/.acme.sh/icodedream.com_ecc/icodedream.com.key
  [Thu Apr 28 21:31:30 CST 2022] The intermediate CA cert is in: /root/.acme.sh/icodedream.com_ecc/ca.cer
  [Thu Apr 28 21:31:30 CST 2022] And the full chain certs is there: /root/.acme.sh/icodedream.com_ecc/fullchain.cer
  [root@VM-0-14-centos ~]# 
  ```

  - CPOY证书到Nginx
  
  ```bash
  acme.sh --installcert -d icodedream.com --ecc \
        --key-file /usr/local/nginx/cert/icodedream.com.key \
        --fullchain-file /usr/local/nginx/cert/fullchain.cer \
        --reloadcmd "docker exec -it nginx service nginx reload"
  ```
  ![](../_images/_linux/dnspod-4.png)

  - 更新 acme.sh
  
  目前由于 acme 协议和 letsencrypt CA 都在频繁的更新, 因此 acme.sh 也经常更新以保持同步。

  1. 开启自动升级

  ```bash
  # cme.sh  --upgrade  --auto-upgrade

  [root@VM-0-14-centos ~]# acme.sh  --upgrade  --auto-upgrade
  [Thu Apr 28 23:05:49 CST 2022] Already uptodate!
  [Thu Apr 28 23:05:49 CST 2022] Upgrade success!
  [root@VM-0-14-centos ~]# 
  ```

  2. 关闭自动更新

  ```bash
  acme.sh --upgrade  --auto-upgrade  0
  ```
  3. HTTPS安全与兼容性配置指南
   
      https://blog.myssl.com/https-security-compatibility-best-practices/#a