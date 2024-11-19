 

### 1.先查看系统版本

```bash
[root@yzn ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core) 
[root@yzn ~]# ssh -V
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017
```

### 2.下载openssh+openssl安装包（这里openssh升级到9.6，openssl升级到1.1.1）

```bash
[root@yzn opt]# wget https://www.openssl.org/source/openssl-1.1.1w.tar.gz  
[root@yzn opt]# wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.6p1.tar.gz 
[root@yzn opt]# ll
-rw-r--r-- 1 root root 1857862 Dec 18 23:06 openssh-9.6p1.tar.gz
-rw-r--r-- 1 root root 9893384 Dec  4 22:39 openssl-1.1.1w.tar.gz 
[root@yzn opt]# mv /opt/open* /app && cd /app 
[root@yzn app]# ll 
open*
-rw-r--r-- 1 root root 1857862 Dec 18 23:06 openssh-9.6p1.tar.gz
-rw-r--r-- 1 root root 9893384 Dec  4 22:39 openssl-1.1.1w.tar.gz 
[root@yzn app]# tar -xvzf openssh-9.6p1.tar.gz 
[root@yzn app]# tar -xvzf openssl-1.1.1w.tar.gz
```

### 3.安装telnet-server服务端（防止卸载旧ssh版本后，无法登陆服务器）

```bash
#telnet运行依赖xinetd，需要一并安装
root@yzn ~]# yum -y install telnet-server.x86_64 xinetd.x86_64 运行telnet服务
[root@yzn ~]# systemctl enable telnet.socket
[root@yzn ~]# systemctl start telnet.socket
[root@yzn ~]# systemctl start xinetd[root@yzn ~]# systemctl status telnet.socket      #查看telnet运行状态[root@yzn ~]# netstat -nltp                       #查看telnet端口23在不在
```

### 4.Linux系统中默认不允许root账户telnet远程登录，这里需要移除配置文件，保证root账户能够远程登录

```bash
[root@yzn ~]# mv /etc/securetty /etc/securetty.bak
```

### 5.使用telnet协议测试远程连接服务器(检查服务器防火墙是否开放23端口访问)

![](https://img-blog.csdnimg.cn/direct/5dfa4578f6db4b0f8c5b1bd7d694763c.png)

###  6.卸载旧版本的ssh

```bash
[root@yzn ~]# yum -y remove openssh[root@yzn ~]# rm -rf /etc/ssh/*
```

### 7.安装编译工具依赖包

```bash
[root@yzn ~]# yum -y install zlib* pam-* gcc make
```

### 8.编译安装openssl

```bash
[root@yzn ~]# cd /app
[root@yzn app]# ll
drwxr-xr-x  7 tomcat tomcat   20480 Jan 24 11:04 openssh-9.6p1-rw-r--r--  1 root   root   1857862 Jan 24 10:25 openssh-9.6p1.tar.gzdrwxrwxr-x 19 root   root      4096 Jan 24 10:59 openssl-1.1.1w-rw-r--r--  1 root   root   9893384 Jan 24 10:25 openssl-1.1.1w.tar.gz 
[root@yzn app]# cd openssl-1.1.1w
[root@yzn openssl-1.1.1w]# ./config --prefix=/usr/local/openssl   
[root@yzn openssl-1.1.1w]# make && make install                    #编译安装 #配置软件链接（如果提示***已存在，将参数换成-b即可）
[root@yzn openssl-1.1.1w]# ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
[root@yzn openssl-1.1.1w]# ln -s /usr/local/openssl/lib/libssl.so.1.1 /usr/lib64/libssl.so.1.1
[root@yzn openssl-1.1.1w]# ln -s /usr/local/openssl/lib/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
```

### 9.查看openssl是否安装成功

```bash
[root@yzn app]# openssl version
OpenSSL 1.1.1w  11 Sep 2023
```

###  10.升级ssh

```bash
[root@yzn ~]# cd /app/openssh-9.6p1
[root@yzn openssh-9.6p1]# ./configure --prefix=/usr/ --sysconfdir=/etc/ssh  --with-openssl-includes=/usr/local/openssl/include --with-ssl-dir=/usr/local/openssl   --with-zlib   --with-md5-passwords   --with-pam 
#如果找不到openssl lib，执行：
[root@yzn openssh-9.6p1]# ./configure --prefix=/usr/ --sysconfdir=/etc/ssh  --with-openssl-includes=/usr/local/openssl/include --with-ssl-dir=/usr/local/openssl   --with-zlib   --with-md5-passwords  CCFLAGS="-I /usr/lib/openssl/openssl/include" LDFLAGS="-L /usr/lib/openssl/openssl/lib64"  --with-pam [root@yzn openssh-9.6p1]# make && make install
复制配置文件
[root@yzn openssh-9.6p1]# cp -a contrib/redhat/sshd.init /etc/init.d/sshd
[root@yzn openssh-9.6p1]# chmod u+x /etc/init.d/sshd
```

### 11.配置/etc/ssh/sshd\_config文件（允许root远程登录）

```bash
[root@yzn ~]# vim /etc/ssh/sshd_config  #在配置内增加下面两行，或取消注释PasswordAuthentication yes     PermitRootLogin yes     添加ssh开机自启
[root@yzn ~]# chkconfig --add sshd
[root@yzn ~]# chkconfig sshd on   重启sshd服务
[root@yzn ~]# systemctl restart sshd
[root@yzn ~]# systemctl status sshd.....active (running)....... 查看版本
[root@yzn ~]# ssh -V
OpenSSH_9.6p1, OpenSSL 1.1.1w  11 Sep 2023
[root@yzn ~]# netstat -nltp
Active Internet connections (only servers)Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    tcp        0      0 0.0.0.0:81              0.0.0.0:*               LISTEN      1344/nginx: master  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      26436/sshd: /usr/sb tcp6       0      0 :::22                   :::*                    LISTEN      26436/sshd: /usr/sb
```

12.测试连接

13.卸载telnet-server

```bash
[root@yzn ~]# yum -y remove telnet-server
```

本文转自 <https://blog.csdn.net/Yzn970608/article/details/135814769>，如有侵权，请联系删除。