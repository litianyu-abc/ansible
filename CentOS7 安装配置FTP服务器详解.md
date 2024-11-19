 

CentOS7 安装配置FTP服务器详解
--------------------

1、FTP简介
-------

ftp（File Transfer [Protocol](https://so.csdn.net/so/search?q=Protocol&spm=1001.2101.3001.7020)文件传输协议）是基于TCP/IP 协议的应用层协议，用于文件的传输，包括ftp服务器（或服务端）和ftp客户端

ftp客户端与服务器创建网络连接，请求登录服务器，登录成功后，就可以进行文件传输，主要包括下载文件和上传文件两种操作

2、关闭防火墙
-------

为了避免不必要的麻烦，我们先关闭防火墙和selinux，等搭建成功之后再开启防火墙和相应的端口

```bash
[root@centos7 ~]# systemctl status firewalld.service        # 查看防火墙状态
[root@centos7 ~]# systemctl stop firewalld.service          # 停止防火墙服务
[root@centos7 ~]# systemctl disable firewalld.service       # 关闭防火墙开启自启动
# 把文件中的SELINUX=enforcing 改为SELINUX=disabled
[root@centos7 ~]# vim /etc/selinux/config          
[root@centos7 ~]# setenforce 0                              # 使修改马上生效
```

3、安装FTP软件包
----------

在CentOS7中，采用yum来安装ftp软件包，包括ftp服务器和ftp客户端

> 查看是否已经安装了vsftpd

```bash
# 如果没有返回任何结果，表示没有安装；如果返回文件包名，这表示已经安装了该服务；
[root@centos7 ~]# rpm -qa|grep vsftpd
vsftpd-3.0.2-29.el7_9.x86_64               # 代表已安装
[root@centos7 ~]# vsftpd -version
vsftpd: version 3.0.2                      # 代表已安装
[root@centos7 ~]# rpm -e vsftpd            # 卸载vsftpd
# 再次检查
[root@centos7 ~]# rpm -qa|grep vsftpd
[root@centos7 ~]# vsftpd -version
bash: vsftpd: 未找到命令...
```

> 开始安装vsftpd

*   安装ftp服务器

```bash
# 如果已经安装，再次执行yum就会把软件包升级到最新版本
[root@centos7 ~]# yum install -y vsftpd
```

*   安装ftp客户端

```bash
[root@centos7 ~]# yum install -y ftp lftp
```

*   设置为开机自动启动服务

```bash
[root@centos7 ~]# systemctl enable vsftpd.service 
```

*   启动vsftpd服务

```bash
# ftp服务器的服务名是vsftpd，相关的操作如下：
[root@centos7 ~]# systemctl start  vsftpd.service      # 启动服务
systemctl stop  vsftpd.service        # 停止服务
systemctl restart vsftpd.service      # 重启服务
systemctl status vsftpd.service       # 查看服务状态
systemctl enable vsftpd.service       # 设置开机自启动vsftpd服务
systemctl disable vsftpd.service      # 禁用开机自启动vsftpd服务
```

4、新建用户和FTP目录
------------

> ftpuser是你为该ftp服务创建的用户名，/data/ftp/ftpuser为ftp服务器访问路径

*   新建FTP目录并授权

```bash
# 创建文件目录
mkdir -p /data/ftp/ftpuser
# 为该目录配置权限
chmod -R 755 /data/ftp/ftpuser
```

如果我们直接使用`useradd -d ftpuser`，则新建的用户是可以登录系统的，这样会给FTP服务器带来安全隐患

因此我们为了不让FTP用户登录系统，就必须为FTP用户统一创建一个不能登录系统的shell，这一行的命令只运行一次即可，后面新建用户就不需要执行了

```bash
echo /usr/bin/nologin>>/etc/shells
```

*   新建ftp组及用户

```bash
# 新建用户组ftp
groupadd ftp
# 指定用户主目录：/data/ftp/ftpuser -M
# 指定用户的shell： -s /usr/bin/nologin
# 新建用户ftpuser，并且设置不支持ssh系统登录，只能登录ftp服务器
# -g 用户组； -d 指定家目录； -s 不能登陆系统； -M 不创建家目录
useradd -g ftp -d /data/ftp/ftpuser -M -s /usr/bin/nologin ftpuser

==============================================================
# 如果要恢复ftpuser用户的ssh登录（可登入CentOS7系统），执行下面的语句即可
usermod -s /bin/bash ftpuser
```

*   设置密码

```bash
# echo "新密码" | passwd --stdin 用户名
echo "ftppassword" | passwd --stdin ftpuser
```

*   新建FTP用户可写目录

```bash
# 由于/data/ftp/ftpuser的用户是root，其它用户都没有写的权限
# 所以要在该目录下新建一个目录用于文件的上传下载
mkdir -p /data/ftp/ftpuser/upload
chown ftpuser:ftp /data/ftp/ftpuser/upload
chmod 755 /data/ftp/ftpuser/upload
```

5、配置ftp服务器
----------

> 备份配置文件

```bash
# 防止后期配置文件出错后无法还原
[root@centos7 ~]# cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
[root@centos7 ~]# cd /etc/vsftpd/
[root@centos7 ssh]# ll
......
-rw------- 1 root root 5116 6月  10 2021 vsftpd.conf
-rw------- 1 root root 5116 8月  15 22:05 vsftpd.conf.backup
......
```

> 编辑配置文件

`vim /etc/vsftpd/vsftpd.conf`

打开vsftpd.conf文件，全选删除（Esc+gg+dG）文件内容，然后用下面的配置进行替换

```bash
# 是否开启匿名用户，匿名都不安全，默认NO
anonymous_enable=NO
# 允许本机账号登录FTP
# 这个设定值必须要为YES时，在/etc/passwd内的账号才能以实体用户的方式登入我们的vsftpd主机
local_enable=YES
# 允许账号都有写操作
write_enable=YES
# 本地用户创建文件或目录的掩码
# 意思是指：文件目录权限：777-022=755，文件权限：666-022=644
local_umask=022
# 进入某个目录的时候，是否在客户端提示一下
dirmessage_enable=YES
# 当设定为YES时，使用者上传与下载日志都会被记录起来
xferlog_enable=YES
# 日志成为std格式
xferlog_std_format=YES
# 上传与下载日志存放路径
xferlog_file=/var/log/xferlog
# 开放port模式的20端口的连接
connect_from_port_20=YES
# 关于系统安全的设定值：
# ascii_download_enable=YES(NO)
# 如果设定为YES，那么client就可以使用ASCII格式下载档案
# 一般来说，由于启动了这个设定项目可能会导致DoS的攻击，因此预设是NO
# ascii_upload_enable=YES(NO)
# 与上一个设定类似的，只是这个设定针对上传而言，预设是NO
ascii_upload_enable=NO
ascii_download_enable=NO
# 通过搭配能实现以下几种效果：
# ①当chroot_list_enable=YES，chroot_local_user=YES时，在/etc/vsftpd/chroot_list文件中列出的用户，可以切换到其他目录；未在文件中列出的用户，不能切换到其他目录
# ②当chroot_list_enable=YES，chroot_local_user=NO时，在/etc/vsftpd/chroot_list文件中列出的用户，不能切换到其他目录；未在文件中列出的用户，可以切换到其他目录
# ③当chroot_list_enable=NO，chroot_local_user=YES时，所有的用户均不能切换到其他目录
# ④当chroot_list_enable=NO，chroot_local_user=NO时，所有的用户均可以切换到其他目录
# 限制用户只能在自己的目录活动
chroot_local_user=YES
chroot_list_enable=NO
chroot_list_file=/etc/vsftpd/chroot_list
# 可以更改ftp的端口号，使用默认值21
# listen_port=60021
# 监听ipv4端口，开了这个就说明vsftpd可以独立运行，不用依赖其他服务
listen=NO
# 监听ipv6端口
listen_ipv6=YES
# 打开主动模式
port_enable=YES
# 启动被动式联机(passivemode)
pasv_enable=YES
# 被动模式端口范围：注意：linux客户端默认使用被动模式，windows 客户端默认使用主动模式。在ftp客户端中执行"passive"来切换数据通道的模式。也可以使用"ftp -A ip"直接使用主动模式。主动模式、被动模式是有客户端来指定的
# 上面两个是与passive mode使用的port number有关，如果您想要使用64000到65000这1000个port来进行被动式资料的连接，可以这样设定
# 这两项定义了可以同时执行下载链接的数量
# 被动模式起始端口，0为随机分配
pasv_min_port=64000
# 被动模式结束端口，0为随机分配
pasv_max_port=65000
# 文件末尾添加
# 这个是pam模块的名称，我们放置在/etc/pam.d/vsftpd，认证用
pam_service_name=vsftpd
# 使用允许登录的名单，在/etc/vsftpd/user_list文件中添加新建的用户ftpuser
userlist_enable=YES
# 限制允许登录的名单，前提是userlist_enable=YES，其实这里有点怪,禁止访问名单在/etc/vsftpd/ftpusers
userlist_deny=NO
# 允许限制在自己的目录活动的用户拥有写权限
# 不添加下面这个会报错：500 OOPS: vsftpd: refusing to run with writable root inside chroot()
allow_writeable_chroot=YES
# 当然我们都习惯支持TCP Wrappers的啦
# Tcp wrappers ： Transmission Control Protocol (TCP) Wrappers 为由 inetd 生成的服务提供了增强的安全性
tcp_wrappers=YES
# FTP访问目录
local_root=/data/ftp/ftpuser
```

> 允许新建用户登录FTP

`vim /etc/vsftpd/user_list`

将新建用户ftpuser添加到/etc/vsftpd/user\_list文件末尾

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f294ac61d7d834b8e0d8a0997f1c970.png#pic_center)

注意：

这个是允许登录ftp的名单，一行一个用户，不能把多个写到一行

> 修改用户切换目录的权限

用户切换目录的规则设定在`/etc/vsftpd/vsftpd.conf`文件中，如果有用户有权限切换目录，则在`/etc/vsftpd/chroot_list`文件中添加即可（一行一个用户）

例如：

```bash
# 在chroot_list中添加新建用户ftpuser
# 第一步：打开文件
vim /etc/vsftpd/chroot_list
# 第二步：添加ftpuser用户
ftpuser
# 第三步：保存文件并退出
```

6、重启并配置防火墙
----------

```bash
systemctl enable firewalld.service                           # 重启防火墙开机自启动
systemctl restart firewalld.service                          # 重启防火墙服务
firewall-cmd --version                                       # 查看防火墙版本
firewall-cmd --list-all       					             # 查看已开放的端口
firewall-cmd --permanent --zone=public --add-service=ftp     # 防火墙开通ftp服务
firewall-cmd --permanent --zone=public --add-port=21/tcp     # 开通ftp服务21命令控制端口
# 主动模式下数据传输端口等于命令控制端口-1 ======> 21 - 1 = 20
# 开通ftp服务主动模式的20数据传输端口
firewall-cmd --permanent --zone=public --add-port=20/tcp    
# 开通ftp服务被动模式的数据端口范围
firewall-cmd --permanent --zone=public --add-port=64000-65000/tcp 
firewall-cmd --reload                            # 刷新防火墙，重新载入
# 设置关闭SELinux对ftp的限制
setsebool -P ftpd_full_access on
sed -i s#enforcing#disabled#g /etc/sysconfig/selinux
setenforce 0 && getenforce
getenforce
```

7、重启FTP服务
---------

```bash
systemctl restart vsftpd.service
```

**至此，FTP其实就已经搭建成功，可以登录了！**

8、访问测试
------

> 查看[IP](https://so.csdn.net/so/search?q=IP&spm=1001.2101.3001.7020)地址

`ip addr`

注意：

*   云服务器的ip地址为`公网ip地址`
*   虚拟机的ip地址为NAT模式下的`固定ip地址`，下图用的就是固定ip

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e259d0a98a031540a109958596d611b2.png#pic_center)

> 新建测试文件

```bash
# 进入之前设置好的路径
[root@centos7 ~]# cd /data/ftp/ftpuser/upload
# 新建测试文件，然后保存退出
[root@centos7 upload]# vim 测试_20220712.txt
[root@centos7 upload]#
```

> 浏览器访问测试

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59ced15f9eea9db47a2bb7ec3bbecb54.png#pic_center)

> 终端访问测试

*   ftp命令

```bash
[root@centos7 ~]# ftp 192.168.10.110
Connected to 192.168.10.110 (192.168.10.110).
220 (vsFTPd 3.0.2)
Name (192.168.10.110:root): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,10,110,251,154).
150 Here comes the directory listing.
drwxr-xr-x    2 1002     50           4096 Aug 15 14:26 upload
226 Directory send OK.
ftp> cd upload
250 Directory successfully changed.
ftp> ls
227 Entering Passive Mode (192,168,10,110,250,171).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              12 Aug 15 14:26 测试_20220712.txt
226 Directory send OK.
ftp> exit
221 Goodbye.
[root@centos7 ~]#
```

*   lftp命令

```bash
# 格式：lftp 用户名:密码@ftp地址:传送端口（默认21）
[root@centos7 ~]# lftp ftpuser:ftppassword@192.168.10.110:21
lftp ftpuser@192.168.10.110:~> ls
drwxr-xr-x    2 1002     50           4096 Aug 15 14:26 upload
lftp ftpuser@192.168.10.110:/> cd upload/
lftp ftpuser@192.168.10.110:/upload> ls
-rw-r--r--    1 0        0              12 Aug 15 14:26 测试_20220712.txt
lftp ftpuser@192.168.10.110:/upload> exit
[root@centos7 ~]# 
```

9、拓展知识
------

> 文件加密传输配置

*   下载并安装openssl

```bash
yum -y install openssl openssl-devel
```

*   首先生成CA私钥文件并改变权限

```bash
openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048
chmod 400 /etc/pki/CA/private/cakey.pem
```

*   生成自签证书

```bash
openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 365
```

*   为证书和私钥文件单独创建一个隐藏目录并设置访问权限

```bash
mkdir -p /etc/vsftpd/.ssl
chmod 400 /etc/vsftpd/.ssl
cd /etc/vsftpd/.ssl
```

*   生成私钥并设置访问权限

```bash
openssl genrsa -out vsftpd.key 2048
chmod 400 vsftpd.key
```

*   生成请求签署文件

```bash
openssl req -new -key vsftpd.key -out vsftpd.csr
```

*   签发证书

```bash
touch /etc/pki/CA/index.txt
echo 01 | sudo tee /etc/pki/CA/serial
openssl ca -in /etc/vsftpd/.ssl/vsftpd.csr -out /etc/vsftpd/.ssl/vsftpd.crt -days 365 
```

*   vsftp配置文件末尾添加以下信息 `vim /etc/vsftpd/vsftpd.conf`

```bash
#开启ssl证书功能
ssl_enable=YES
#匿名用户相关配置
allow_anon_ssl=YES
force_anon_data_ssl=YES
force_anon_logins_ssl=YES
#不重用SSL会话
require_ssl_reuse=NO
#在数据传输时采用证书加密
force_local_data_ssl=YES
# 在登录时采用证书加密
force_local_logins_ssl=YES
# ssl的版本一名字是tsl
ssl_tlsv1=YES
# ssl的版本二名字是ssl
ssl_sslv2=YES
# ssl的版本三名字是ssl
ssl_sslv3=YES
# 设置SSL密码等级
ssl_ciphers=HIGH
# ssl的证书文件存放位置
rsa_cert_file=/etc/vsftpd/.ssl/vsftpd.crt
# ssl的私钥文件存放位置
rsa_private_key_file=/etc/vsftpd/.ssl/vsftpd.key
# 启用SSL调试，将openSSL连接记录到VSFTPD日志文件中
debug_ssl=YES
```

*   重启FTP服务

```bash
systemctl restart vsftpd.service
```

