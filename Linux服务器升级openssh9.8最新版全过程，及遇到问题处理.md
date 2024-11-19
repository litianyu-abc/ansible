 

前言：由于2024年7月1日，openssh发布了最新版9.8，所以[服务器](https://so.csdn.net/so/search?q=%E6%9C%8D%E5%8A%A1%E5%99%A8&spm=1001.2101.3001.7020)需要升级一下，特此做个详细记录：

由于下载最新版openssh9.8，需要将openssl也一并进行升级

### 一、下载openssh最新版本与openssl对应版本：

openssh最新版本下载地址：[https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable//](https://link.zhihu.com/?target=https%3A//cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/ "https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable//")

![](https://img-blog.csdnimg.cn/direct/0524a21e7ffb455f9f0a7893f00388f1.png)

openssl对应版本下载地址（这下载OpenSSL 1.1.1v就可以）：

[Release OpenSSL 1.1.1v · openssl/openssl · GitHub](https://github.com/openssl/openssl/releases/tag/OpenSSL_1_1_1v "Release OpenSSL 1.1.1v · openssl/openssl · GitHub")

![](https://img-blog.csdnimg.cn/direct/ddc584cb7815454f9a665d35925e1bc8.png)

下载之后，请上传到需要升级的服务器的路径下

### 二、openssl安装：

#### 2-1、安装前检查：

```bash
# 查看CentOS系统版本信息cat /etc/redhat-release# 查看openssl版本信息openssl version# 查看openssh的版本信息ssh -V
```

#### 2-2、安装依赖

```csharp
# 安装相关的依赖项，如有遗漏再次安装yum -y install gcc pam-devel zlib-devel openssl-devel net-tools
```

#### 2-3、安装下载的openssl

由于openssh9.8p1要求openssl版本大于等于1.1.1,因此需要升级[安装openssl](https://so.csdn.net/so/search?q=%E5%AE%89%E8%A3%85openssl&spm=1001.2101.3001.7020)。

##### ①、解压openssl-1.1.1v.tar.gz

```cobol
# 将openssl-1.1.1v解压到/usr/local目录下Tar zxvf /home/test/openssl-1.1.1v.tar.gz –C /usr/local/# 查看解压后目标目录情况ll /usr/local/ | grep openssl# 进入openssl-1.1.1v目录下cd /usr/local/openssl-1.1.1v/
```

##### ②、openssl准备及安装

```bash
# 创建安装目录mkdir /opt/openssl# 安装前查看openssl详细版本信息openssl version #编译# 配置编译和安装过程，"--prefix=" 选项配置安装目录./config --prefix=/opt/openssl# 构建程序以及所需的指令和依赖关系make# 安装编译好的openssl-1.1.1wmake install # 备注：以上三步必须没有出现报错（error），才可以继续下一步，全部三步无报错才能视为安装成功
```

##### ③、更新lib文件

```cobol
# 检查openssl-1.1.1v所需要的函数库ldd /opt/openssl/bin/openssl# 添加openssl-1.1.1v的库文件路径到ld.so.confecho "/opt/openssl/lib" >> /etc/ld.so.conf# 更新系统函数库库 ldconfig --verbose# 绝对路径查看openssl版本ldd /opt/openssl/bin/openssl
```

##### ④、更新bin文件

```bash
# 查看旧版本的openssl命令路径which openssl# 重命名为openssl.oldmv /bin/openssl /bin/openssl.old   #重命名openssl文件# 使用软连接的方式更新openssl命令ln -s /opt/openssl/bin/openssl /bin/openssl# openssl命令查看版本呢openssl version
```

三、openssh9.8安装

3-1、解压下载的安装包，卸载原来包

```crystal
tar zxvf openssh9.7p1.tar.gz # 卸载openssh的rpm包for i in $(rpm -qa | grep openssh);do rpm -e $i --nodeps;done
```

3-2、进入解压文件后，开始编译

```cobol
# 配置编译和安装过程，"--prefix=" 配置安装目录，"--sysconfdir=" 配置文件路径，"--with-ssl-dir=" openssl的安装路径 ./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-pam --with-ssl-dir=/opt/openssl --with-md5-passwords --mandir=/usr/share/man --with-zlib=/usr/local/zlib --without-hardening # 构建程序以及所需的指令和依赖关系make# 安装编译好的openssh9.8p1make install # 备注：以上三步必须没有出现报错（error），才可以继续下一步，全部三步无报错才能视为安装成功
```

3-3、复制并修改启动sshd.init脚本

```cobol
# 从源码目录下复制sshd.init到/etc/init.d/cp contrib/redhat/sshd.init /etc/init.d/ ## 查看并修改SSHD的新路径,将新的openssh安装路径更新cat /etc/init.d/sshd.init | grep SSHDsed -i "s/SSHD=\/usr\/sbin\/sshd/SSHD=\/usr\/local\/openssh\/sbin\/sshd/g" /etc/init.d/sshd.initcat /etc/init.d/sshd.init | grep SSHD ## 查看并修改ssh-keygen的新路径，将新的ssh-keygen安装路径更新cat -n /etc/init.d/sshd.init | grep ssh-keygensed -i "s#/usr/bin/ssh-keygen -A#/usr/local/openssh/bin/ssh-keygen -A#g" /etc/init.d/sshd.initcat -n /etc/init.d/sshd.init | grep ssh-keygen
```

3-4、修改配置文件

```bash
# 开启允许X11转发echo 'X11Forwarding yes' >> /etc/ssh/sshd_config # 开启允许密码验证echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config 
```

3-5、启动openssh，并设置开机启动

```cobol
# 复制ssh的相关命令cp -arp /usr/local/openssh/bin/* /usr/bin/# 启动sshd服务/etc/init.d/sshd.init# 查看版本ssh –V# 添加开机启动chmod +x /etc/rc.d/rc.localecho “/etc/init.d/sshd.init start” >> /etc/rc.d/rc.local  
```

如果启动不成功，通过命令查看，是什么原因导致的

```cobol
#通过 systemctl status sshd查看sshd是否正常启动systemctl status sshd如果执行上边命令也不可以，那么需要执行：yum install openssh-server #执行后，如果显示activing或者dead没有正常启动，那么需要执行命令，查看具体异常原因： tail -f /var/log/messages #根据日志情况，结果看到原因是22端口被占用导致无法正常启动（图当时没截）
```

所以需要将22端口kill掉，特别注意，执行下边这个命令后，你连接的这台服务器会马上连接不上了！！！，前边都执行没有问题，才可以执行，执行过几分钟后，再来链接发现正常链接上了：

```cobol
fuser -k 22/tcp     #执行时需特别注意 #如果怕执行上诉命令后连接不上，可以设置一个linux定时，操作方法如下： 使用cron的方法：编辑当前用户的crontab文件： crontab -e 添加以下行来检查SSH服务并在它停止时重启： * * * * * pgrep sshd > /dev/null || /usr/sbin/sshd 这个crontab条目每分钟都会检查sshd进程是否运行。如果不运行，它将尝试重启SSH服务。
```

再次连接，发现可以正常链接上了，并通过命令查看是否为最新版本：

![](https://img-blog.csdnimg.cn/direct/c63dd99fb51a4fd5ad2505b48d0ac15d.png)

结束！

本文转自 <https://blog.csdn.net/qq18346342939/article/details/140149193>，如有侵权，请联系删除。