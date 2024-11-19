# Openssh版本升级（Centos6.7）

> 实现前提
> 公司服务器需要进行安全测评，扫描漏洞的设备扫出了关于 openssh 漏洞，主要是因为 openssh的当前版本为5.3，版本低了，而yum最新的openssh也只是5.3，没办法只能到 rpm 官网找新的包，找到最新的是 6.4，然后通过 yum localinstall 升级了，但是安全部门反映还存在 openssh 漏洞，没办法只能去openssh官网找最新的release，安装版本是 7.7！
> 升级原因
> 7.4以下版本openssh版本存在严重漏洞：
> 1.OpenSSH 远程权限提升漏洞(CVE-2016-10010)
> 2.OpenSSH J-PAKE授权问题漏洞(CVE-2010-4478)
> 3.Openssh MaxAuthTries限制绕过漏洞(CVE-2015-5600)
> OpenSSL>=1.0.1可以不用升级OpenSSL
> 升级前操
> 为避免升级过程中ssh异常，造成连接断开而无法继续操作服务器，故先留条后路，部署telnet；操作如下：
> \# yum -y install telnet-server* telnet
> \# vi /etc/xinetd.d/telnet (将其中disable字段的yes改为no以启用telnet服务)
> \# mv /etc/securetty /etc/securetty.old #允许root用户通过telnet登录
> \# service xinetd start
> \# chkconfig xinetd on
> \# 部署完毕后，用telnet测试登陆服务器；
> 升级操作
> 笔者将升级内容做成了脚本，在服务器上执行以下脚本，即可实现openssh的版本升级，脚本如下：
> \# cat upte_openssh.sh

```shell
#!/bin/bash
# by kazihuo.
wget https://openbsd.hk/pub/OpenBSD/OpenSSH/portable/openssh-7.7p1.tar.gz
mv /etc/ssh /etc/ssh.old 
mv /etc/init.d/sshd /etc/init.d/sshd.old
yum -y remove openssh openssh-server openssh-clients openssh-askpass
install -v -m700 -d /var/lib/sshd 
chown -v root:sys /var/lib/sshd 
groupadd -g 50 sshd 
useradd -c 'sshd PrivSep' -d /var/lib/sshd -g sshd -s /bin/false -u 50 sshd
tar -axf openssh-7.7p1.tar.gz
cd openssh-7.7p1
yum -y install pam-devel gcc pam-devel zlib-devel
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords --with-pam --with-zlib --with-openssl-includes=/usr --with-privsep-path=/var/lib/sshd
make && make install
install -v -m755 contrib/ssh-copy-id /usr/bin
install -v -m644 contrib/ssh-copy-id.1 /usr/share/man/man1 
install -v -m755 -d /usr/share/doc/openssh-7.7p1 
install -v -m644 INSTALL LICENCE OVERVIEW README* /usr/share/doc/openssh-7.7p1 
ssh -V
echo 'X11Forwarding yes' >> /etc/ssh/sshd_config 
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
cp -p contrib/redhat/sshd.init /etc/init.d/sshd 
chmod +x /etc/init.d/sshd 
chkconfig --add sshd 
chkconfig sshd on 
service sshd restart
```

版本验证
\# ssh -V
OpenSSH_7.7p1, OpenSSL 1.0.2k-fips 26 Jan 2017



# Linux - 升级OpenSSH 以及 OpenSSL的思路

## 背景

1. 安全审计问题（更新版本来升级漏洞），必须升级系统SSH服务版本。
2. 个人爱好。

## 问题

1. 查看最新版本的OpenSSH软件包里的说明文件(INSTALL)，它告诉我们需要依赖其他软件包，例如OpenSSL，PAM，ZLIB等。
2. 如果升级小版本也可以修复主要漏洞（特别的人对“大版本漏洞少”存在误解）：yum update openssl -y
3. 操作系统本身有很多应用依赖于当前的OpenSSL编译的，如果你升级大版本，可能很多程序不能用，跨大版本是不可行的。
4. 如果你需要升级大版本，请升级操作系统版本。
5. 单独安装一个OpenSSL，然后依赖此来升级SSH服务。

## 准备信息

1. [下载安装包：openssh-7.9p1.tar.gz](https://openbsd.hk/pub/OpenBSD/OpenSSH/portable/)
2. 依赖Zlib Zlib 1.1.4 or 1.2.1.2 or greater (earlier 1.2.x versions have problems):
   [DownLoad Old](http://www.gzip.org/zlib/) [DownLoad New](https://zlib.net/)
3. 依赖libcrypto (LibreSSL or OpenSSL >= 1.0.1 < 1.1.0)
   [DownLoad LibreSSL](http://www.libressl.org/) or [DownLoad OpenSSL](http://www.openssl.org/source/)
4. 依赖FIPS
   [DownLoad FIPS: openssl-fips-2.0.16.tar.gz](https://www.openssl.org/source/)

注意：1.0.x版本最新的是openssl-1.0.2q.tar.gz
注意：说明文档里面说不支持1.1.x版本的OpenSSL

**补充：openssl与openssl-fips有什么区别**
联邦信息处理标准（Federal Information Processing Standards，FIPS）是一套描述文件处理、加密算法和其他信息技术标准（在非军用*机构和与这些机构合作的*承包商和供应商中应用的标准）的标准。

## [安装zlib](https://www.553668.com/default/index/url?u=aHR0cDovL3d3dy5idWJ1a28uY29td2l6Oi8vb3Blbl9kb2N1bWVudD9ndWlkPTVlMmM0Mzk5LTAxNzgtNDJjZC05NGY0LTRkZTJlZDlmZDBhYiZrYmd1aWQ9JnByaXZhdGVfa2JndWlkPWY4NDUxYzQyLTczZGItNGNhMy04Y2NkLWMwY2JiMzg4ZDdmNw==)

## 安装PAM

PAM(Pluggable Authentication Modules，可插拔认证模块)用于提供安全控制。

```shell
# 操作系统也已经自带了PAM，版本满足要求
pam-1.1.8-22.el7.x86_64
# 安装开发包
yum install pam-devel -y
```

## 安装OpenSSL

### 编译安装FIPS模块

```shell
export FIPSDIR=/opt/fips-2.0.16
tar -xvf openssl-fips-2.0.16.tar.gz
cd openssl-fips-2.0.16
./config
make
make install
```

注意：在编译前先设定环境变量FIPSDIR，这是用于指定FIPS模块的安装目录，这是fips软件特有的安装特性。软件编译时会检测该环境变量是否存在。若不指定，默认会安装在/usr/local/ssl/fips2.0目录。

### 编译安装OpenSSL

```shell
tar zxvf openssl-1.0.2q.tar.gz
cd openssl-1.0.2q
mkdir /opt/openssl-1.0.2q_20190131
./config --prefix=/opt/openssl-1.0.2q_20190131 --openssldir=/opt/openssl-1.0.2q_20190131/openssl fips --with-fipsdir=/opt/fips-2.0.16 zlib-dynamic shared -fPIC
make depend
make
make test
make install
/opt/openssl-1.0.2q_20190131/bin/openssl version -a

# 说明如下
--prefix：指定openssl的安装目录。按本例中的安装方式，安装完成后该目录下会包含bin(含二进制程序)、lib(含动态库文件)、include/openssl(含报头文件)及openssl(--openssldir选项指定的)这些子目录。
--openssldir：指定openssl文件的安装目录。按本例中的安装方式，安装完成后该目录下会包括certs(存放证书文件)、man(存放man文件)、misc(存放各种脚本)、private(存放私钥文件)这些子目录及openssl.cnf配置文件。
fips：集成FIPS模块。
--with-fipsdir：指向FIPS模块的安装目录位置。
zlib-dynamic：编译支持zlib压缩/解压缩，让openssl加载zlib动态库。该选项只在支持加载动态库的操作系统上才支持。这是默认选项。
shared：除了静态库以外，让openssl(在支持的平台上)也编译生成openssl动态库。
-fPIC：将openssl动态库编译成位置无关(position-independent)的代码。
# 安装完成后，将OpenSSL的库文件目录添加到/etc/ld.so.conf文件中，并加载到系统内存缓存中：
echo ‘/opt/openssl-1.0.2q_20190131/lib‘ >> /etc/ld.so.conf
ldconfig
```

## 安装OpenSSH

### 编译安装

从openssl官网下载源码包openssh-7.9p1.tar.gz，将其安装到/opt/openssh-7.9.p1_20190131目录下。
将openssh安装到专门的目录，这是为了避免与操作系统自带的openssh造成不必要的冲突，增加复杂度。

```shell
[root@node1 opt]# cd /opt
[root@node1 opt]# tar -xvf openssh-7.9p1.tar.gz
[root@node1 opt]# cd openssh-7.9p1/
[root@node1 openssh-7.9p1]# ./configure --prefix=/opt/openssh-7.9.p1_20190131 --with-ssl-dir=/opt/openssl-1.0.2q_20190131 --with-pam --with-tcp-wrappers
[root@node1 openssh-7.9p1]# make
[root@node1 openssh-7.9p1]# make install

# 说面
--prefix：指定安装目录
--with-ssl-dir=DIR：指向LibreSSL/OpenSSL库的安装目录的所在路径。
--with-pam：启用PAM支持。根据OpenSSH的安装说明，如果在编译时启用了PAM，那么在安装完成后，也必须在sshd服务的配置文件sshd_config中启用它(使用UsePAM指令)。
```

根据OpenSSH的安装说明，如果有启用PAM，那么就需要手工安装一个给sshd程序使用的PAM配置文件，否则安装好OpenSSH后你可能会无法使用密码登录系统。
在编译时，我使用 --with-pam 选项启用了对PAM的支持，但是，编译OpenSSH时并没有编译选项让你指定PAM配置文件的位置，那么我们要怎么提供这个配置文件呢？

事实上，OpenSSH有另外一个编译选项**--with-pam-service=\*name\***可以指定PAM服务名，它的默认值是sshd。
而操作系统自带的PAM软件默认将所有PAM配置文件都放置在**/etc/pam.d**目录下。
结合这两个信息，就可确定OpenSSH的PAM配置文件应为**/etc/pam.d/sshd**文件。而这个文件原来就有了，所以我们不用额外手工创建一个。

```shell
# 设置PATH路径
[root@node1 profile.d]# echo ‘export PATH=/opt/openssh-7.9.p1_20190131/bin:/opt/openssh-7.9.p1_20190131/sbin:$PATH‘ >> /etc/profile.d/path.sh
# 加载PATH
[root@node1 profile.d]# . /etc/profile.d/path.sh
# 验证版本
[root@node1 profile.d]# ssh -V
OpenSSH_7.9p1, OpenSSL 1.0.2q-fips  20 Nov 2018
```

### 修改配置

```shell
# 可以先看一下原有的sshd服务(属于openssh-server软件包)都有哪些配置文件：
rpm -ql openssh-server | grep -i --color etc
/etc/pam.d/sshd       # pam配置文件
/etc/ssh/sshd_config  # sshd服务的配置文件
/etc/sysconfig/sshd   # 启动脚本文件？

# 参照系统原有的配置文件修改我们编译出来的的sshd_config配置文件：
vi /opt/openssh-7.9.p1_20190131/etc/sshd_config
# 放开注释或添加如下内容
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication yes
```

### 开机启动

```shell
# 查看当前系统自启动服务列表
[root@node1 init.d]# systemctl list-unit-files |grep sshd
anaconda-sshd.service                         static
sshd-keygen.service                           static
sshd.service                                  enabled
sshd@.service                                 static
sshd.socket                                   disabled
[root@node1 init.d]#

# 关闭当前系统上的老版本SSH服务
systemctl disable sshd.service

# 创建新服务的配置文件：/usr/lib/systemd/system/ssh.service，内容如下
[Unit]
Description=OpenSSH daemon
After=network.target sshd-keygen.service

[Service]
Type=sample # 这里需要设置成sample或者删除这一项。
ExecStart=/opt/openssh7.9.p1_20190131/sbin/sshd

[Install]
WantedBy=multi-user.target

# 添加到系统自启动列表里面
systemctl daemon-reload
systemctl enable ssh.service

# 从自启动列表中删除
systemctl disable ssh.service

# 启动 关闭 重启服务
systemctl stop ssh
systemctl start ssh
systemctl restart ssh

# 注意，连接的时候需要关闭SeLinux，关于如何添加Selinux策略待学习。
setenforce 0
```





# OpenSSH如何升级到最新版本

# 前言

[官网](https://www.openssh.com/portable.html)

[安装说明](https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/INSTALL)

[下载 | FTP](https://www.openssh.com/portable.html#ftp)

[下载 | RSYNC](https://www.openssh.com/portable.html#rsync)

[下载 | HTTP](https://www.openssh.com/portable.html#http)

# 步骤

## 升级脚本

具体的内容请查看脚本内容

```shell
#!/bin/bash

## 查看现有的ssh的版本并升级到最新版本
cd /opt
ssh -V
openssl version
yum update openssh -y

## 安装启动并配置telnet服务 | 防止ssh升级失败无法访问服务器
yum install -y telnet-server* telnet xinetd
systemctl enable xinetd.service
systemctl enable telnet.socket
systemctl start telnet.socket
systemctl start xinetd.service
echo ‘pts/0‘ >>/etc/securetty
echo ‘pts/1‘ >>/etc/securetty
echo ‘pts/2‘ >>/etc/securetty

## 升级ssh
yum install  -y gcc gcc-c++ glibc make autoconf openssl openssl-devel pcre-devel  pam-devel
yum install  -y pam* zlib*
wget -c https://openbsd.hk/pub/OpenBSD/OpenSSH/portable/openssh-8.1p1.tar.gz
wget -c https://ftp.openssl.org/source/openssl-1.0.2r.tar.gz
tar xfz openssh-8.1p1.tar.gz
tar xfz openssl-1.0.2r.tar.gz
mv /usr/bin/openssl /usr/bin/openssl_bak
mv /usr/include/openssl /usr/include/openssl_bak
cd /opt/openssl-1.0.2r
./config shared && make && make install
echo $?
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/ssl/include/openssl /usr/include/openssl
echo "/usr/local/ssl/lib" >> /etc/ld.so.conf
/sbin/ldconfig
openssl version
cd /opt/openssh-8.1p1
chown -R root.root /opt/openssh-8.1p1
cp -r  /etc/ssh /tmp/
rm -rf /etc/ssh
./configure --prefix=/usr/ --sysconfdir=/etc/ssh  --with-openssl-includes=/usr/local/ssl/include --with-ssl-dir=/usr/local/ssl   --with-zlib   --with-md5-passwords   --with-pam  && make && make install
echo $?
 
cat > /etc/ssh/sshd_config <<EOF
PermitRootLogin yes
AuthorizedKeysFile      .ssh/authorized_keys
UseDNS no
Subsystem       sftp    /usr/libexec/sftp-server
EOF
grep "^PermitRootLogin"  /etc/ssh/sshd_config
cat /tmp/ssh/sshd_config |grep -v ‘#‘ |grep -v ‘^$‘
cp -a contrib/redhat/sshd.init /etc/init.d/sshd
cp -a contrib/redhat/sshd.pam /etc/pam.d/sshd.pam
chmod +x /etc/init.d/sshd
chkconfig --add sshd
systemctl enable sshd
mv  /usr/lib/systemd/system/sshd.service  /opt/
mv  /usr/lib/systemd/system/sshd.socket  /opt/
chkconfig sshd on
service sshd restart
openssl version
ssh -V
```

## 关闭telnet服务

自测后如果没有问题的话,自行把telnet服务关闭

```shell
systemctl disable xinetd
systemctl disable telnet.socket
systemctl stop xinetd.service
systemctl stop telnet.socket
```

## 效果如下

![Linux——OpenSSH如何升级到最新版本](https://www.553668.com/default/index/img?u=aHR0cDovL2ltYWdlMS5idWJ1a28uY29tL2luZm8vMjAyMDA2LzIwMjAwNjEwMTMzNjA3MDY1Mzg3LnBuZw==)

[Linux——OpenSSH如何升级到最新版本](https://www.553668.com/default/index/url?u=aHR0cDovL3d3dy5idWJ1a28uY29tL2luZm9kZXRhaWwtMzU4Nzk0MC5odG1s)



# Linux openssl1.0.2k升级openssl1.1.1e版本教程

1.查看openssl版本

```shell
[root@localhost ~]# openssl version
OpenSSL 1.0.2k-fips  26 Jan 2017
```

2.下载指定版本的openssl软件
在下面网址：https://www.openssl.org/source/下载 后面的版本号可以换

```shell
[root@localhost ~]# wget https://www.openssl.org/source/openssl-1.1.1e.tar.gz
```

3.编译安装

```shell
cd /opt
tar xfz openssl-1.1.1e.tar.gz -C /opt/
[root@localhost opt]# cd openssl-1.1.1e/
./config shared zlib
make && make install
```

4.配置

```shell
[root@localhost ~]# mv /usr/bin/openssl /usr/bin/openssl.bak
[root@localhost ~]# mv /usr/include/openssl /usr/include/openssl.bak
[root@localhost ~]#  find / -name openssl
[root@localhost ~]# ln -s /usr/local/bin/openssl /usr/bin/openssl
[root@localhost ~]# ln -s /usr/local/include/openssl /usr/include/openssl
[root@localhost ~]# echo "/usr/local/lib64/" >> /etc/ld.so.conf
[root@localhost ~]# ldconfig
[root@localhost ~]# openssl version -a
```

5.验证

```shell
[root@node2 openssl-1.1.1e]# openssl version
OpenSSL 1.1.1e  28 May 2019
```





















































































