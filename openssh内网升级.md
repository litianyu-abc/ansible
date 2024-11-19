一、安装telnet
==========

1.上传如下的rpm安装包
-------------

http://mirrors.163.com/centos/7.6.1810/os/x86\_64/Packages/telnet-0.17-64.el7.x86\_64.rpm
http://mirrors.163.com/centos/7.6.1810/os/x86\_64/Packages/telnet-server-0.17-64.el7.x86\_64.rpm
http://mirrors.163.com/centos/7.6.1810/os/x86\_64/Packages/xinetd-2.3.15-13.el7.x86\_64.rpm

2.安装telnet
----------

rpm -ivh xinetd-2.3.15\-13.el7.x86\_64.rpm
rpm \-ivh telnet-server-0.17\-64.el7.x86\_64.rpm
rpm \-ivh telnet-0.17\-64.el7.x86\_64.rpm

3.配置telnet（这步必须确保操作正确，否则telnet无法连接）
-----------------------------------

```shell
cat <<EOF>> /etc/securetty
pts/0
pts/1
pts/2
pts/3
pts/4
EOF
tail -n 5 /etc/securetty
```

4.启动telnet
----------

```shell
systemctl enable xinetd
systemctl start xinetd
systemctl status xinetd
systemctl enable telnet.socket
systemctl start telnet.socket
systemctl status telnet.socket
```

5.防火墙放行（也可直接关闭防火墙）
------------------

firewalld-cmd --permanent --zone=public --add-port=23/tcp
firewalld\-cmd --reload

二、安装pam-devel
=============

1.先查看是否安装了pam及其版本
-----------------

rpm -qa | grep pam

2.若已经安装了pam，则安装同一个版本的pam-devel
------------------------------

例如rpm -qa | grep pam搜出的rpm版本是pam-1.1.8-18.el7.x86\_64
那么就需要安装pam-devel-1.1.8-18.el7.x86\_64.rpm
在这个页面中的搜索框中搜一下就有了：http://rpm.pbone.net/index.php3/stat/3/limit/2/srodzaj/1/dl/40/search
ftp://ftp.pbone.net/mirror/ftp.scientificlinux.org/linux/scientific/7.4/x86\_64/os/Packages/pam-devel-1.1.8-18.el7.x86\_64.rpm
rpm -ivh pam-devel-1.1.8-18.el7.x86\_64.rpm

3.若未安装pam，则下载最新版的安装即可
---------------------

http://mirrors.aliyun.com/centos/7/os/x86\_64/Packages/pam-1.1.8-22.el7.x86\_64.rpm
http://mirrors.aliyun.com/centos/7/os/x86\_64/Packages/pam-devel-1.1.8-22.el7.x86\_64.rpm
rpm -ivh pam-1.1.8-22.el7.x86\_64.rpm 
rpm -ivh pam-devel-1.1.8-22.el7.x86\_64.rpm

三、安装zlib
========

1.上传如下的安装包
----------

http://www.zlib.net/fossils/zlib-1.2.11.tar.gz

2.解压并安装
-------

tar -zxvf zlib-1.2.11.tar.gz
cd zlib\-1.2.11
./configure --prefix=/usr/local/zlib-1.2.11 -share
make && make install
ln -s /usr/local/zlib-1.2.11 /usr/local/zlib

3.将zlib动态函数库加载到高速缓存中
--------------------

echo "/usr/local/zlib-1.2.11/lib" >> /etc/ld.so.conf
ldconfig \-v

四、安装openssl
===========

1.上传如下的安装包
----------

http://distfiles.macports.org/openssl/openssl-1.0.2m.tar.gz

2.解压并安装
-------

```shell
tar zxvf openssl-1.0.2m.tar.gz
cd openssl\-1.0.2m
./config shared zlib-dynamic --prefix=/usr/local/openssl-1.0.2m --with-zlib-lib=/usr/local/zlib-1.2.11/lib --with-zlib-include=/usr/local/zlib-1.2.11/include
make
make test   #若有报错时一定要先解决，不要往下执行
make install
ln -s /usr/local/openssl-1.0.2m /usr/local/openssl
```

3.将openssl动态函数库加载到高速缓存中
-----------------------

echo "/usr/local/openssl-1.0.2m/lib" >> /etc/ld.so.conf   
ldconfig \-v

4.将openssl工具集路径加入到path路径中
-------------------------

echo 'export PATH=/usr/local/openssl/bin:$PATH' >> /etc/profile
source /etc/profile

5.查看openssl的版本号，以验正是否安装正确
-------------------------

openssl version -a 

五、卸载旧版openssh
=============

1.如果旧版是rpm安装的话
--------------

rpm -qa | grep openssh | xargs rpm -e --nodeps
rm -rf /etc/ssh/ssh\_host\*

2.如果旧版是编译安装的话
-------------

找到之前的安装包，在里面执行：
make uninstall
rm -rf /etc/ssh/ssh\_host\*

六、安装新版openssh
=============

1.上传如下的安装包
----------

https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.0p1.tar.gz

2.解压并安装
-------

tar -xzvf openssh-8.0p1.tar.gz
cd openssh\-8.0p1
./configure --prefix=/usr/ --sysconfdir=/etc/ssh --with-pam --with-md5-passwords --with-tcp-wrappers --with-ssl-dir\=/usr/local/openssl --with-zlib=/usr/local/zlib --mandir=/usr/share/man   
make && make install

3.进行必要的配置
---------

```shell
echo "PermitRootLogin yes" >> /etc/ssh/sshd\_config
sed -i '/UsePAM no/c\\UsePAM yes' /etc/ssh/sshd\_config
cp -p contrib/redhat/sshd.init /etc/init.d/sshd 
chmod +x /etc/init.d/sshd 
sed -i '/^Subsystem/c\\Subsystem sftp /usr/libexec/sftp-server' /etc/ssh/sshd\_config
sed -i '/^SELINUX=enforcing/c\\SELINUX=disabled' /etc/selinux/config
setenforce 0
```

4.设置sshd开机自启动
-------------

chkconfig --add sshd 
chkconfig sshd on 
chkconfig \--list sshd 

5.重启sshd并查看版本
-------------

systemctl restart sshd
ssh -V

本文转自 <https://www.cnblogs.com/jipinglong/p/16611152.html>，如有侵权，请联系删除。