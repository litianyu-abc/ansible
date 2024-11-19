 

在日常开发中，经常会需要升级服务器漏洞，记录一下linux升级openssh相关，服务器版本为centos7.8，升级有两种方案，一种是可以上互联网环境，一种是内网环境，我这边因为是内网环境，只能进行离线安装包式升级。

1、准备工作

下载好需要的安装包，我用到的openssh-9.6p1.tar.gz、openssl-3.2.0.tar.gz、pam-devel-1.1.1-20.el6.x86\_64.rpm

2、升级准备

开启telnet：  
1、检查是否安装了telnet/xinted  
    rpm -qa telnet  
    rpm -qa xinted  
2、没有安装的话进行安装  
    yum -y install xinetd  
      
3、启动telnet  
    systemctl start telnet.socket  
    systemctl start xinetd 或者service xinetd start

查看是否启动：

service xinetd status  
4、查看端口是否监听(监听表示启动成功)  
    netstat -tnl|grep 23  
5、如果是为了升级ssh，需要检查防火墙是否  
    firewall-cmd --list-all  
    添加开放防火墙

  
安装升级ssl  
1、安装包复制到服务器上（ssl1.1.1）  
2、解压安装ssl  
    tar zxvf ssl.tar.gz  
    ./config shared --prefix=/usr/local/ssl && make && make install  
3、#备份老的openssl  
mv /usr/bin/openssl /usr/bin/openssl\_bak  
mv /usr/include/openssl /usr/include/openssl\_bak

\# 建立我们编译好的软连接  
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl  
 ln -s /usr/local/ssl/include/openssl /usr/include/openssl  
 #确认 版本是否是对的  
  /usr/bin/openssl version

\# 添加动态链接库，并生效  
echo "/usr/local/ssl/lib" >> /etc/ld.so.conf  
ldconfig

  
      
安装升级ssh  
1、将需要的依赖包放到服务器上（ssh、ssl包）  
2、升级ssh  
    ./configure --prefix=/usr --sysconfdir=/etc/ssh --with-openssl-includes=/usr/local/openssl/include --with-ssl-dir=/usr/local/openssl --without-openssl-header-check --with-zlib --with-md5-password --with-pam && make && make install

3、报错提示configure: error: PAM headers not found  
    缺少pam-devel 包  
    手动安装pam包  rpm -ivh pam-devel-1.1.8-20.el6.x86\_64.rpm --nodeps --force  
    或者使用 yum -y install yam-devel  
4、执行2 报错  libpam missing

5、执行2  报警告make: \[check-config\] Error 1 (ignored)    
    chmod 600 /etc/ssh/ssh\_host\_rsa\_key  
    chmod 600 /etc/ssh/ssh\_host\_ecdsa\_key  
    chmod 600 /etc/ssh/ssh\_host\_ed25519\_key

6、设置openssh-8.0p1的环境信息（在openssh解压目录内）  
    install -v -m755 contrib/ssh-copy-id /usr/bin  
    install -v -m644 contrib/ssh-copy-id.1 /usr/share/man/man1  
    install -v -m755 -d /usr/share/doc/openssh-8.0p1  
    install -v -m644 INSTALL LICENCE OVERVIEW README\* /usr/share/doc/openssh-8.0p1  
      
7、设置启动脚本  
在openssh-8.0编译目录下  
cp -a contrib/redhat/sshd.init /etc/init.d/sshd  
cp -a contrib/redhat/sshd.pam /etc/pam.d/sshd.pam  
chmod +x /etc/init.d/sshd

8、备份旧的，重新生成新的，要不然影响systemctl  
mv  /usr/lib/systemd/system/sshd.service  ./  
chkconfig --add sshd  
systemctl start sshd #启动服务  
systemctl enable sshd #设置开机启动  
chkconfig sshd on  
/usr/sbin/sshd #没有结果是最好的结果

重启sshd  
systemctl restart sshd

8、先不关闭。检查是否能登录  
    不能登录查看getenforce 是否开启，然后关闭  
    setenforce 0

openssh下载链接，链接为上面升级版本，其他版本自行更改：https://www.linuxfromscratch.org/blfs/view/svn/postlfs/openssh.html

本文转自 <https://blog.csdn.net/star950117/article/details/139861677>，如有侵权，请联系删除。