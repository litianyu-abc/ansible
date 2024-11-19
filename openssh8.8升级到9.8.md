一、概述
----

openssh最近爆出安全漏洞，需要升级到9.8以上版本，生产环境当前版本是8.8，自己搭建的虚拟机是默认的7.4，于是决定先从7.4升级到8.8，再从8.8升级到9.8，完成漏洞修复

二、升级步骤
------

复制

```plain
1、查看版本
[root@localhost openssh-8.8p1]# ssh -V
OpenSSH_8.8p1, OpenSSL 1.0.2k-fips  26 Jan 2017

2、下载安装包
cd /usr/local/src
wget https://www.zlib.net/zlib-1.3.1.tar.gz
wget https://www.openssl.org/source/openssl-3.2.1.tar.gz
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.8p1.tar.gz

3、配置备份
cp -rf /etc/ssh /etc/ssh.bak
cp -rf /usr/bin/openssl /usr/bin/openssl.bak
cp -rf /etc/pam.d /etc/pam.d.bak
cp -rf /usr/lib/systemd/system /usr/lib/systemd/system.bak
rm -rf /etc/ssh/*

4、解压
cd /usr/local/src/
tar -zxvf zlib-1.3.1.tar.gz
tar -zxvf openssl-3.2.1.tar.gz
tar -zxvf openssh-9.8p1.tar.gz

5、安装zlib
cd /usr/local/src/zlib-1.3.1
./configure --prefix=/usr/local/src/zlib
make -j 4 && make test && make install

6、安装openssl
需要额外安装依赖
yum install -y perl-IPC-Cmd
cd /usr/local/src/openssl-3.2.1
./config --prefix=/usr/local/src/openssl
make -j 4 && make install

mv /usr/bin/openssl /usr/bin/oldopenssl
ln -s /usr/local/src/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/src/openssl/lib64/libssl.so.3 /usr/lib64/libssl.so.3
ln -s /usr/local/src/openssl/lib64/libcrypto.so.3 /usr/lib64/libcrypto.so.3

7、安装openssh
cd /usr/local/src/openssh-9.8p1/
./configure --prefix=/usr/local/src/ssh --sysconfdir=/etc/ssh --with-pam --with-ssl-dir=/usr/local/src/openssl --with-zlib=/usr/local/src/zlib 
make -j 4 && make install
#查看目录版本
/usr/local/src/ssh/bin/ssh -V
#复制新ssh文件
cp -rf /usr/local/src/openssh-9.8p1/contrib/redhat/sshd.init /etc/init.d/sshd
cp -rf /usr/local/src/ssh/sbin/sshd /usr/sbin/sshd
cp -rf /usr/local/src/ssh/bin/ssh /usr/bin/ssh
cp -rf /usr/local/src/ssh/bin/ssh-keygen /usr/bin/ssh-keygen

8、启动服务
手动更新/etc/ssh/sshd_config
#重启sshd服务
/etc/init.d/sshd restart
#查看服务运行状态
/etc/init.d/sshd status
#添加开机启动
chkconfig --add sshd
#查看升级后ssh版本
[root@localhost ssh]# ssh -V
OpenSSH_9.8p1, OpenSSL 3.2.1 30 Jan 2024
```

三、报错处理
------

复制

```plain
启动sshd后会报错：
"/sbin/restorecon:  lstat(/etc/ssh/ssh_host_dsa_key.pub) failed:  No such file or directory"

查看资料后知晓
从官方文档中可查，dsa从openssh 9.0就不再使用，rsa从openssh 9.3开始就不再使用

解决办法更改/etc/init.d/sshd
将此行注释掉 #/sbin/restorecon /etc/ssh/ssh_host_dsa_key.pub

重新加载启动即可
systemctl daemon-reload
systemctl restart sshd
```

本文转自 <https://blog.51cto.com/u_13236892/11335364>，如有侵权，请联系删除。