**一、问题背景**

目前部署在centos操作系统的服务器的系统，经过单位网络安全部门的漏扫测试比较容易扫描的一个漏洞（CVE-2023-38408）修复的建议是Openssh版本过低需要升级至9.6p1版本，但是安装的过程中其它依赖也需要进行安装比如Openssl和zlib。

（**安装前需要注意本人安装过程未安装xinetd-2.3.15和telnet-0.17-65，这两个服务主要是保障在安装openssh过程中一旦出现中断现象，服务器可以通过telnet方式登录，稳妥起见可以自行搜索安装这两个服务**）

![Centos7 离线升级Openssh-9.6p1版本_服务器](https://s2.51cto.com/images/blog/front/202401/f5d5be157d34ad834df000731127913fc6d8e4.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

**二、下载安装包**

需要先下载安装包后，**上传到服务器/usr/local目录下**

①Openssh-9.6p1下载地址： [https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/](https://www.openssl.org/source/old/1.1.1/index.html)

![Centos7 离线升级Openssh-9.6p1版本_安装包_02](https://s2.51cto.com/images/blog/front/202401/766e0a372356cd72d3339270f2232b1ba378b4.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

②Openssl-1.1.1q下载地址： [https://www.openssl.org/source/old/1.1.1/index.html](https://www.openssl.org/source/old/1.1.1/index.html)

![Centos7 离线升级Openssh-9.6p1版本_服务器_03](https://s2.51cto.com/images/blog/front/202401/739d4120843b33a694731477b14cb99b8907a1.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

③zlib1.3下载地址： [http://www.zlib.net/](http://www.zlib.net/)

![Centos7 离线升级Openssh-9.6p1版本_重启_04](https://s2.51cto.com/images/blog/front/202401/13dda33441c2e31615b22192ce952687fc2f7e.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

④pam-1.1.8-23和pam-devel-1.1.8-23下载地址： [https://rpmfind.net/linux/rpm2html/search.php?query=pam&submit=Search+...&system=&arch=](https://rpmfind.net/linux/rpm2html/search.php?query=pam&submit=Search+...&system=&arch=)

![Centos7 离线升级Openssh-9.6p1版本_安装包_05](https://s2.51cto.com/images/blog/front/202401/87e3574237a2ce50885825e2b2d1d75840770b.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

![Centos7 离线升级Openssh-9.6p1版本_服务器_06](https://s2.51cto.com/images/blog/front/202401/57338592742e4c2c8393692baa6ab748e57401.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

**三、安装步骤**

①安装OPenssl1.1.1q依赖

```shell
#解压安装包，安装包放置路径（/usr/local）
tar -zxf openssl-1.1.1q.tar.gz

#进入安装目录
cd openssl-1.1.1q

#添加路径配置进行初始化
./config -Wl,-rpath=/usr/lib64 --prefix=/usr/local/openssl --openssldir=/usr/local/openssl --libdir=/usr/lib64

#执行安装
make -j 4 && make install

#备份旧openssl版本，创建软连接
mv /usr/bin/openssl /usr/bin/openssl.bak
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl

#验证新版本
openssl version
```

如查看ssl版本时，提示 /bin/sh: gcc: command not found 错误，需要安装gcc依赖，如果没有网络的情况下请下载安装包解压安装，有网络的情况下可通过yum方式安装

```shell
yum install gcc gcc-c++
```

②安装zlib1.3依赖（**注意自己下载的版本是1.3.1的话解压等命令的版本也要修改为1.3.1**）

```shell
#解压安装包，安装包放置路径（/usr/local）
tar xf zlib-1.3.tar.gz

#进入安装目录下
cd zlib-1.3

#添加prefix路径
./configure --prefix=/usr/local/zlib

#编译执行安装 
make
make check
make install

#写入动态库的路径
echo "/usr/local/zlib/lib/" >> /etc/ld.so.conf
ldconfig -v

#查看软连接
ll /lib64/libz.*

#进入编译目录下
cd /usr/local/zlib/lib/

#复制执行文件到/lib64目录下
cp libz.so.1.3 /lib64/libz.so.1.3

#进入lib64目录下创建软连接
cd /lib64
ln -snf libz.so.1.3  /lib64/libz.so
ln -snf libz.so.1.3  /lib64/libz.so.1
```

③安装pam-1.1.8-23和pam-devel-1.1.8-23依赖

```shell
#进入安装包放置目录
cd /usr/local
#放置rpm包到/usr/local路径下，并执行安装命令
rpm -Uvh pam-1.1.8-23.el7.x86_64.rpm
rpm -Uvh pam-devel-1.1.8-23.el7.x86_64.rpm
```

④安装openssh-9.6p1

```shell
#查看SELinux状态（如果SELinux status参数为enabled即为开启状态）
/usr/sbin/sestatus -v

#关闭SELinux
setenforce 0

#修改/etc/selinux/config 文件
vim /etc/selinux/config
#将SELINUX=enforcing改为SELINUX=disabled

#重启机器(注意重启前查看是否还有未配置的)
reboot

重启服务器后重新连接上去

#备份openssh
mkdir /etc/ssh/bak
cp /etc/ssh/ssh* /etc/ssh/bak && cp /etc/ssh/m* /etc/ssh/bak
cp /etc/pam.d/sshd ~

#卸载openssh
rpm -e `rpm -qa | grep openssh` --nodeps

#进入/usr/local解压安装包
tar -xzvf openssh-9.6p1.tar.gz
#进入安装目录
cd openssh-9.6p1
#初始化安装
./configure --prefix=/usr/ --sysconfdir=/etc/ssh --with-pam --with-md5-passwords --with-tcp-wrappers --with-ssl-dir=/usr/local/openssl --with-zlib=/usr/local/zlib --mandir=/usr/share/man
#执行安装
make && make install

#授权
chmod 600 /etc/ssh/ssh_host_rsa_key
chmod 600 /etc/ssh/ssh_host_ecdsa_key
chmod 600 /etc/ssh/ssh_host_ed25519_key

#配置
cp -p contrib/redhat/sshd.init /etc/init.d/sshd
chmod +x /etc/init.d/sshd
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
sed -i '/UsePAM no/c\UsePAM yes' /etc/ssh/sshd_config
sed -i '/^Subsystem/c\Subsystem sftp /usr/libexec/sftp-server' /etc/ssh/sshd_config
sed -i '/^SELINUX=enforcing/c\SELINUX=disabled' /etc/selinux/config
setenforce 0

#查看版本
sshd -v
#测试sshd能否登陆
/usr/sbin/sshd -p 22
```

![Centos7 离线升级Openssh-9.6p1版本_重启_07](https://s2.51cto.com/images/blog/front/202401/f340264600dc2c083f43735f5b7fbe5675a876.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=,x-oss-process=image/resize,m_fixed,w_1184/format,webp)

如果在测试sshd能否登录提示如上图错误，则执行赋权  chmod 600 /etc/ssh/ssh\_host\_ecdsa\_key 同样的报错哪个就这样赋哪个文件的权限

```shell
#设置sshd开机自启动
chkconfig --add sshd
chkconfig sshd on
chkconfig --list sshd

#重启服务
rm -rf /usr/lib/systemd/system/sshd.service
systemctl daemon-reload
systemctl restart sshd
systemctl status sshd
```

如上步骤全部完成即可升级Openssh9.6p1版本

本文转自 <https://blog.51cto.com/u_15514004/9417210?articleABtest=0>，如有侵权，请联系删除。