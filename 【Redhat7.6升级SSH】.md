 

#### Redhat7.6升级SSH

*   [一、Redhat7.6配置网络yum源](#Redhat76yum_1)
*   *   [1、删除redhat7.0系统自带的yum软件包](#1redhat70yum_3)
    *   [2、选择阿里云自行下载所需要的软件包。](#2_8)
    *   *   [2.1 三大网路源地址](#21__9)
        *   [2.2 下载相关rpm包，确保能上外网](#22_rpm_16)
    *   [3、根据依赖项安装](#3_28)
    *   [4、下载repo配置文件](#4repo_35)
    *   [5、下载KEY文件：](#5KEY_39)
    *   *   [5.1 直接下载](#51__40)
        *   [5.2 vim /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7文件，内容如下](#52_vim_etcpkirpmgpgRPMGPGKEYCentOS7_45)
    *   [6、清理yum缓存并重启服务器](#6yum_78)
*   [二、升级ssl和ssh](#sslssh_129)
*   *   [1、查看操作系统情况](#1_130)
    *   [2、下载SSH和SSL安装包](#2SSHSSL_136)
    *   [3、安装Telnet。避免卸载ssh后无法远程登录](#3Telnetssh_142)
    *   [4、用telnet登录并卸载旧版本SSH](#4telnetSSH_156)
    *   [5、安装依赖包](#5_161)
    *   [6、升级ssl](#6ssl_165)
    *   *   [6.1 解压源码包](#61__166)
        *   [6.2 进入源码包执行config文件](#62_config_170)
        *   [6.3 编译安装](#63__178)
        *   [6.4 配置软连接](#64__182)
        *   [6.5 查看是否安装成功](#65__188)
    *   [7、升级SSH](#7SSH_193)
    *   *   [7.1 解压源码包](#71__194)
        *   [7.2 进入源码包目录编译安装](#72__198)
        *   [7.3 复制配置文件](#73__202)
        *   [7.4 添加SSH开机自启动](#74_SSH_208)
        *   [7.5 查看sshd运行状态](#75_sshd_213)
    *   [8、查看升级结果](#8_219)
    *   [9、卸载telnet-server包](#9telnetserver_224)
*   [三、问题处理](#_228)
*   *   [1、当使用 SSH 登录 Linux 云主机时，即便正确输入了密码，在命令行或 secure 日志中也会出现类似如下错误信息，导致连接失败](#1_SSH__Linux__secure__229)
    *   *   [1.1 报错内容](#11__230)
        *   [1.2 错误原因](#12__235)
        *   [1.3 处理办法](#13__239)
    *   [2、将openssl升级至3.x找不到libssl.so.3和libcrypto.so.3](#2openssl3xlibsslso3libcryptoso3_247)
    *   *   [2.1 报错内容](#21__248)
        *   [2.2 报错原因](#22__252)
        *   [2.3 处理办法](#23__256)
    *   [3、SSH文件权限问题，启动 systemctl restart sshd报权限错误](#3SSH_systemctl_restart__sshd_264)

一、Redhat7.6配置网络yum源
-------------------

redhat系统安装好尽管默认带有yum，但是redhat的更新包只对注册用户有效（收费）。所以需要更换yum源，用Redhat相近的centos的yum源。

### 1、删除redhat7.0系统自带的yum软件包

```
rpm -qa|grep yum|xargs rpm -e --nodeps   
#不检查依赖，直接删除rpm包
```

### 2、选择阿里云自行下载所需要的软件包。

#### 2.1 三大网路源地址

```
包名会更新，根据当前最新的下载。如有有依赖问题，下载依赖包进行安装。
阿里云网络源地址：https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/
网易163网络源地址：http://mirrors.163.com/ CentOS
网络源地址：http://centos.ustc.edu.cn/centos/
```

#### 2.2 下载相关rpm包，确保能上外网

```
##因版本升级了，找不到对应文件，需进入//mirrors.aliyun.com/centos/7/os/x86_64/Packages找到正确文件
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-3.4.3-168.el7.centos.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-54.el7_8.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/python-urllib3-1.10.2-7.el7.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-langpacks-0.4.2-7.el7.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/yum-rhn-plugin-2.0.1-10.el7.noarch.rpm
wget https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/rpm-4.11.3-45.el7.x86_64.rpm
```

### 3、根据依赖项安装

```
#进入文件下载目录 /boot/downloads/
rpm -ivh python-*
rpm -Uvh rpm-4* --nodeps
rpm -ivh yum-*
```

### 4、下载repo配置文件

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 5、下载KEY文件：

#### 5.1 直接下载

```
cd  /etc/pki/rpm-gpg
wget   http://mirrors.aliyun.com/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7
```

#### 5.2 vim /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7文件，内容如下

```
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1.4.5 (GNU/Linux)

mQINBFOn/0sBEADLDyZ+DQHkcTHDQSE0a0B2iYAEXwpPvs67cJ4tmhe/iMOyVMh9
Yw/vBIF8scm6T/vPN5fopsKiW9UsAhGKg0epC6y5ed+NAUHTEa6pSOdo7CyFDwtn
4HF61Esyb4gzPT6QiSr0zvdTtgYBRZjAEPFVu3Dio0oZ5UQZ7fzdZfeixMQ8VMTQ
4y4x5vik9B+cqmGiq9AW71ixlDYVWasgR093fXiD9NLT4DTtK+KLGYNjJ8eMRqfZ
Ws7g7C+9aEGHfsGZ/SxLOumx/GfiTloal0dnq8TC7XQ/JuNdB9qjoXzRF+faDUsj
WuvNSQEqUXW1dzJjBvroEvgTdfCJfRpIgOrc256qvDMp1SxchMFltPlo5mbSMKu1
x1p4UkAzx543meMlRXOgx2/hnBm6H6L0FsSyDS6P224yF+30eeODD4Ju4BCyQ0jO
IpUxmUnApo/m0eRelI6TRl7jK6aGqSYUNhFBuFxSPKgKYBpFhVzRM63Jsvib82rY
438q3sIOUdxZY6pvMOWRkdUVoz7WBExTdx5NtGX4kdW5QtcQHM+2kht6sBnJsvcB
JYcYIwAUeA5vdRfwLKuZn6SgAUKdgeOtuf+cPR3/E68LZr784SlokiHLtQkfk98j
NXm6fJjXwJvwiM2IiFyg8aUwEEDX5U+QOCA0wYrgUQ/h8iathvBJKSc9jQARAQAB
tEJDZW50T1MtNyBLZXkgKENlbnRPUyA3IE9mZmljaWFsIFNpZ25pbmcgS2V5KSA8
c2VjdXJpdHlAY2VudG9zLm9yZz6JAjUEEwECAB8FAlOn/0sCGwMGCwkIBwMCBBUC
CAMDFgIBAh4BAheAAAoJECTGqKf0qA61TN0P/2730Th8cM+d1pEON7n0F1YiyxqG
QzwpC2Fhr2UIsXpi/lWTXIG6AlRvrajjFhw9HktYjlF4oMG032SnI0XPdmrN29lL
F+ee1ANdyvtkw4mMu2yQweVxU7Ku4oATPBvWRv+6pCQPTOMe5xPG0ZPjPGNiJ0xw
4Ns+f5Q6Gqm927oHXpylUQEmuHKsCp3dK/kZaxJOXsmq6syY1gbrLj2Anq0iWWP4
Tq8WMktUrTcc+zQ2pFR7ovEihK0Rvhmk6/N4+4JwAGijfhejxwNX8T6PCuYs5Jiv
hQvsI9FdIIlTP4XhFZ4N9ndnEwA4AH7tNBsmB3HEbLqUSmu2Rr8hGiT2Plc4Y9AO
aliW1kOMsZFYrX39krfRk2n2NXvieQJ/lw318gSGR67uckkz2ZekbCEpj/0mnHWD
3R6V7m95R6UYqjcw++Q5CtZ2tzmxomZTf42IGIKBbSVmIS75WY+cBULUx3PcZYHD
ZqAbB0Dl4MbdEH61kOI8EbN/TLl1i077r+9LXR1mOnlC3GLD03+XfY8eEBQf7137
YSMiW5r/5xwQk7xEcKlbZdmUJp3ZDTQBXT06vavvp3jlkqqH9QOE8ViZZ6aKQLqv
pL+4bs52jzuGwTMT7gOR5MzD+vT0fVS7Xm8MjOxvZgbHsAgzyFGlI1ggUQmU7lu3
uPNL0eRx4S1G4Jn5
=OGYX
-----END PGP PUBLIC KEY BLOCK-----
```

### 6、清理yum缓存并重启服务器

```
 yum -y makecache
 ## 如果报错，换源
 http://mirrors.aliyuncs.com/centos/%24releasever/os/x86_64/repodata/repomd.xml: [Errno 14] 
```

尝试CentOS-Base.repo内容更改为163的源  
vim /etc/yum.repos.d/CentOS-Base.repo

```
#CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#
[base]
name=CentOS-$7 - Base - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$7&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/7/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$7 - Updates - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$7&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/7/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$7 - Extras - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$7&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/7/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$7 - Plus - 163.com
baseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

二、升级ssl和ssh
-----------

### 1、查看操作系统情况

```
cat /etc/redhat-release 
#输出：Red Hat Enterprise Linux Server release 7.6 (Maipo)
ssh -V 
```

### 2、下载SSH和SSL安装包

```
用wget下载
openssh-9.5p1.tar.gz
openssl-3.0.12.tar.gz
```

### 3、安装Telnet。避免卸载ssh后无法远程登录

```
#安装组件，Telnet运行需要依靠xinetd组件
yum install telnet-server.x86_64 xinetd.x86_64
#启动服务
systemctl enable telnet.socket
systemctl start telnet.socket 
systemctl start xinetd
#查看服务是否启动
systemctl status telnet.socket 
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
```

### 4、用telnet登录并卸载旧版本SSH

```
yum remove openssh -y   //卸载旧版本SSH
rm -rf /etc/ssh/*       //删除原SSH文件
```

### 5、安装依赖包

```
yum -y install zlib* pam-* gcc make
```

### 6、升级ssl

#### 6.1 解压源码包

```
tar -zxvf openssl-3.0.12.tar.gz
```

#### 6.2 进入源码包执行config文件

```
 cd /root/Downloads/openssl-3.0.12
 ./config --prefix=/usr/local/openssl
 #根据安装过程中报的错误，安装依赖包，如perl、IPC/Cmd.pm等
 yum install perl*  //安装Perl命令
 yum install perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker //Can't locate IPC/Cmd.pm pm
```

#### 6.3 编译安装

```
make -j4 && make install
```

#### 6.4 配置软连接

```
ln -b /usr/local/openssl/lib64/libssl.so.3 /usr/lib64/libssl.so.3
ln -b /usr/local/openssl/lib64/libcrypto.so.3 /usr/lib64/libcrypto.so.3
ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
```

#### 6.5 查看是否安装成功

```
openssl version
#输出：OpenSSL 3.0.12 24 Oct 2023 (Library: OpenSSL 3.0.12 24 Oct 2023)即可
```

### 7、升级SSH

#### 7.1 解压源码包

```
tar -zxvf  openssh-9.5p1.tar.gz
```

#### 7.2 进入源码包目录编译安装

```
./configure --prefix=/usr/ --sysconfdir=/etc/ssh  --with-openssl-includes=/usr/local/openssl/include --with-ssl-dir=/usr/local/openssl   --with-zlib   --with-md5-passwords   --with-pam && make -j4&& make install
```

#### 7.3 复制配置文件

```
cd /root/Downloads/openssh-9.5p1/contrib/redhat
cp -a sshd.init /etc/init.d/sshd
chmod u+x /etc/init.d/sshd
```

#### 7.4 添加SSH开机自启动

```
chkconfig --add sshd
chkconfig sshd on
```

#### 7.5 查看sshd运行状态

```
systemctl restart sshd
systemctl status sshd 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c09060d4b7b648bdb36997b681796e68.png)

### 8、查看升级结果

```
ssh -V
#输出：OpenSSH_9.5p1, OpenSSL 3.0.12 24 Oct 2023，升级成功
```

### 9、卸载telnet-server包

```
yum remove telnet-server -y //只删除服务端，不删除客户端
```

三、问题处理
------

### 1、当使用 SSH 登录 Linux 云主机时，即便正确输入了密码，在命令行或 secure 日志中也会出现类似如下错误信息，导致连接失败

#### 1.1 报错内容

```
Permission denied, please try again.
error: Could not get shadow infromation for root.
```

#### 1.2 错误原因

```
该问题通常是由于系统启用了 SELinux 所致。
```

#### 1.3 处理办法

```
编辑 /etc/selinux/config 文件
SELINUX=disabled

临时关闭SELinux
setenforce 0
```

### 2、将openssl升级至3.x找不到libssl.so.3和libcrypto.so.3

#### 2.1 报错内容

```
openssl: error while loading shared libraries: libssl.so.3: cannot open shared object file: No such file or directory
```

#### 2.2 报错原因

```
以前老版本的lib是1.0，没有把/usr/local/openssl/lib64/的libssl.so.3与/usr/lib或/usr/lib64建立软连接
```

#### 2.3 处理办法

```
ln -b /usr/local/openssl/lib64/libssl.so.3 /usr/lib64/libssl.so.3
ln -b /usr/local/openssl/lib64/libcrypto.so.3 /usr/lib64/libcrypto.so.3

ln -b /usr/local/openssl/lib64/libssl.so.3 /usr/lib/libssl.so.3
ln -b /usr/local/openssl/lib64/libcrypto.so.3 /usr/lib/libcrypto.so.3
```

### 3、SSH文件权限问题，启动 systemctl restart sshd报权限错误

```
chmod 600 /etc/ssh/ssh*
systemctl status sshd.service
systemctl restart sshd 
systemctl status sshd.service
```

本文转自 <https://blog.csdn.net/sxsl_lwp/article/details/135019386>，如有侵权，请联系删除。