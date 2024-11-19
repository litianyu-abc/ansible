# centos7离线升级openssh9.8p1

因公司多数环境处于内网中，无法联网更新，本篇内容主要讲内网更新方法。本篇内容虽经过多次实测，但建议参照本篇升级时先在测试环境进行验证。

所需环境：

本地搭建可连接网络的测试环境，供下载相关的依赖包

#### **一、查看当前机器的版本号**

<table><tbody><tr><td><pre data-index="0" class="set-code-show" name="code" style="user-select: auto;">ssh -V</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/010adb4df1bc4161ae229efb62cec18a.png)

#### 二、了解当前最新openssh与对应兼容的openssl版本信息

[OpenSSHthe main OpenSSH page![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://www.openssh.com/](https://www.openssh.com/ "OpenSSH")

以下可以看到最新的是openssh9.8版本，其对应的openssl版本是 >= 1.1.1k


![](https://img-blog.csdnimg.cn/d58694e402124d39892e7a225b22e8b8.png)

#### 三、下载对应的openssh、openssl包

openssh ： [Index of /pub/OpenBSD/OpenSSH/portable/![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/](https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/ "Index of /pub/OpenBSD/OpenSSH/portable/")

openssl ： [/source/old/1.1.1/index.html![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://www.openssl.org/source/old/1.1.1/](https://www.openssl.org/source/old/1.1.1/ "/source/old/1.1.1/index.html")

```shell
wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.8p1.tar.gz
```

#### 四、编译安装dropbear（可选）

dropbear是一款轻量级的ssh工具，安装这个工具主要是防止升级openssh的过程中出现问题后，导致我们无法在连接我们的服务器，保险起见建议安装。

```shell
wget  https://matt.ucc.asn.au/dropbear/dropbear-2024.85.tar.bz2
```

![image-20240703093358337](C:\Users\EDY\AppData\Roaming\Typora\typora-user-images\image-20240703093358337.png)

4. 1解压dropbear文件

```shell
 tar jxvf dropbear-2022.82.tar.bz2
./configure
make  && make install
```

备注1：第一次安装解压会报错

tar (child): bzip2：无法 exec: 没有那个文件或目录

解决：

```shell
yum install bzip2
```

备注2：安装时会报错

[root@k8s-single-node01 dropbear-2024.84]# ./configure
which: no hg in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
checking for gcc... no
checking for cc... no
checking for cl.exe... no
checking for clang... no
configure: error: in `/opt/dropbear-2024.84':
configure: error: no acceptable C compiler found in $PATH
See `config.log' for more details

解决：

```shell
sudo yum install gcc
sudo yum install clang
```

五、离线下载依赖

备注：在有网的服务器上下载

```shell
]# yum install --downloadonly --downloaddir=/opt/ssl  gcc clang bzip2 zlib zlib-devel openssl-devel pam-devel 
```

![image-20240703094221558](C:\Users\EDY\AppData\Roaming\Typora\typora-user-images\image-20240703094221558.png)

六、安装依赖

```shell
rpm -Uvh  *.rpm  --nodeps  --force 

# 加入环境变量
export CPPFLAGS="-I/usr/include/security"
```

七、配置dropbear，生成密钥

```shell
mkdir /opt/dropbear
/usr/local/bin/dropbearkey -t ed25519 -f /etc/dropbear/dropbear_ed25519_host_key
/usr/local/bin/dropbearkey -t rsa -f /etc/dropbear/dropbear_rsa_host_key
/usr/local/bin/dropbearkey -t ecdsa -f /etc/dropbear/dropbear_ecdsa_host_key

```

八、启动dropbear,注意启动后需要开放2222端口访问权限

```shell
/usr/local/sbin/dropbear -p 2222

#查看端口是否启动
# tail -fn300  secure | grep  dropbear      #查看日志
Jul  3 10:00:46 k8s-single-node01 dropbear[80925]: Running in background
Jul  3 10:01:36 k8s-single-node01 dropbear[81836]: Child connection from 192.168.145.1:16633
Jul  3 10:01:36 k8s-single-node01 dropbear[81836]: Failed loading /etc/dropbear/dropbear_ed25519_host_key
Jul  3 10:01:38 k8s-single-node01 dropbear[81836]: Password auth succeeded for 'root' from 192.168.145.1:16633
#看到这里代表登陆成功
[root@k8s-single-node01 log]# ss -tuln | grep 2222
tcp    LISTEN     0      128       *:2222                  *:*                  
tcp    LISTEN     0      128    [::]:2222               [::]:* 
```

九、下载OpenSSL源码：

下载最新版本的OpenSSL源码，或者至少是1.1.1版本：

```shell
wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz
```

解压源码包：

```shell
tar -xzvf openssl-1.1.1k.tar.gz
cd openssl-1.1.1k
./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl shared zlib
make  &&  make  install
mv /usr/bin/openssl /usr/bin/openssl.bak
```

创建软连接

```shell
[root@k8s-single-node01 openssl-1.1.1k]# ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
[root@k8s-single-node01 openssl-1.1.1k]# echo  "/usr/local/openssl/lib" >> /etc/ld.so.conf
[root@k8s-single-node01 openssl-1.1.1k]# ldconfig -v
```

验证

```shell
]# openssl version
OpenSSL 1.1.1k  25 Mar 2021
```

十、安装openssh9.8

```shell
tar -xzvf openssh-9.8p1.tar.gz
cd  openssh-9.8p1
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-ssl-engine=openssl
make && make install
```

备注：编译时可能会报OpenSSL library version... configure: error: OpenSSL >= 1.1.1 required (have "100020bf (OpenSSL 1.0.2k-fips  26 Jan 2017)")

解决

```shell
# 将openssl lib文件路径导入系统,否则编译会提示找不到lib文件
export LDFLAGS="-L/usr/local/openssl/lib"
export CPPFLAGS="-I/usr/local/openssl/include"
```

验证ssh是否会启动异常

```shell
sshd -T
```

需要对三个文件进行授权

```shell
[root@k8s-single-node01 ssh]# chmod 600  /etc/ssh/ssh_host_rsa_key
[root@k8s-single-node01 ssh]# chmod  600  /etc/ssh/ssh_host_ecdsa_key
[root@k8s-single-node01 ssh]# chmod  600  /etc/ssh/ssh_host_ed25519_key
]# sshd -T
```

修改sshd配置文件

```shell
vim  sshd_config
#PermitRootLogin yes  #去掉#注释
service   restart  sshd   重启sshd
```





```
# 从上面链接中，下载 OpenSSH 官网提供的源码并解压
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.8p1.tar.gz
tar -xzf openssh-9.8p1.tar.gz
cd openssh-9.8p1

# 安装构建所需要的依赖项，一般这步省略（基本这些内容服务器都早已经安装）
sudo apt update -y
sudo apt install build-essential libssl-dev zlib1g-dev -y

# 先备份配置文件
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo cp /etc/ssh/ssh_config /etc/ssh/ssh_config.bak

# 配置编译和安装
./configure --prefix=/usr --sysconfdir=/etc/ssh
make
sudo make install

# 重启 SSH 服务
sudo systemctl restart ssh

# 验证安装，看是否是最新版
ssh -V
```



## openssh9.8升级后的优化

```shell
#编辑/etc/pam.d/sshd
vim  /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth


#修改/etc/ssh/sshd_config
UsePAM yes
PasswordAuthentication yes
PermitRootLogin yes

#重启
systemctl  restart  sshd
```































































































































































