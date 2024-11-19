## 一、环境信息

说明：当前环境openssh版本为9.0p1，是从7.4p1版本直接升级上来的，先需要将9.0p1版本升级至9.3p1版本。如下所示，则为源ssh和目标ssh的信息。

| -    | **操作系统**   | **openssh版本** | **openssh安装目录**      | **openssh安装方式** | **openssl版本** | **openssl安装目录**      | **openssl安装方式** |
| ---- | -------------- | --------------- | ------------------------ | ------------------- | --------------- | ------------------------ | ------------------- |
| 源   | Centos7.6.1810 | 9.0p1           | /usr/local/openssh_9     | 编译安装            | 1.1.1           | /usr/local/openssl       | 编译安装            |
| 目标 | Centos7.6.1810 | 9.3p1           | /usr/local/openssh-9.3p1 | 编译安装            | 3.1.0           | /usr/local/openssl-3.1.0 | 编译安装            |

注意：升级后由于加密算法的区别，低版本的SSH工具可能无法连接，建议改用Xshell7或SecureCRT9.0以上版本。

## 二、注意事项

1、 检查防火墙或selinux是否关闭。
2、 新前一定要多开1个或1个以上ssh终端，一旦更新失败当前shell终端是无法操作的，也就无法进行版本回退。
3、 升级前一定要对ssh进行备份，避免更新失败时能回滚。
4、升级前一定要提前在测试环境验证，运行一段时间，确认没有问题才可在生产环境进行更新操作。
5、对于生产环境主机数量比较多时，建议先在1台或几台服务器上更新，运行一段时间，确认没有问题再执行批量更新操作。

总结：对于以上需要注意的内容，建议大家务必重视，小心谨慎总没错的。

## 三、升级步骤

**[OpenSSH9.3源码包资源合集](https://download.csdn.net/download/m0_37814112/88352738)**

https://download.csdn.net/download/m0_37814112/88352738

### 3.1、准备工作

**1、安装依赖**

```sh
yum install -y gcc gcc-c++ perl perl-IPC-Cmd pam pam-devel
```

**2、[下载tar包并解压](https://download.csdn.net/download/m0_37814112/88352738)**

```sh
cd /usr/local/src
wget https://www.openssl.org/source/openssl-3.1.0.tar.gz --no-check-certificate
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.3p1.tar.gz 
wget http://www.zlib.net/zlib-1.2.13.tar.gz
tar axf openssh-9.3p1.tar.gz && tar axf openssl-3.1.0.tar.gz && tar axf zlib-1.2.13.tar.gz
```

### 3.2、备份文件

**1、备份openssl**

**说明：由于之前的openssl是编译安装的，且/usr/bin/openssl文件和/usr/include/openssl目录均是软连接，所以不用备份，后期如果需要回退版本，则可以重新添加软连接即可。**

如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/21012d715f424427ba69defc29c3dbd1.png)

**2、备份openssh**

```sh
mv /etc/ssh /etc/ssh_bak
mkdir /usr/bin/bak
\cp -arpf /usr/bin/{cp,sftp,ssh,ssh-add,ssh-agent,ssh-keygen,ssh-keyscan} /usr/bin/bak/
\cp -arpf /usr/sbin/sshd /usr/sbin/sshd_bak
\cp -arpf /etc/sysconfig/sshd /etc/sysconfig/sshd_bak
\cp -arpf /usr/lib/systemd/system/sshd.service /usr/lib/systemd/system/sshd.service_bak
\cp -arpf /etc/pam.d/sshd.pam /etc/pam.d/sshd.pam_bak
\cp -arpf /etc/pam.d/sshd /etc/pam.d/sshd_bak
```

**说明：如果cp、sftp、ssh、ssh-add、ssh-agent、ssh-keygen、ssh-keyscan等二进制文件是软连接，这里就不需要备份，请直接删除这些软连接，后续如果还原的时候请从这些文件的源路径里拷贝即可。当前环境不是软连接，所以对这些二进制文件进行备份。**

### 3.3、编译安装zlib

```sh
cd zlib-1.2.13
./configure --prefix=/usr/local/zlib-1.2.13 && make -j 4 && make install
```

### 3.4、编译安装openssl

**1、编译安装**

```sh
cd openssl-3.1.0
./config --prefix=/usr/local/openssl-3.1.0
make -j 4 && make install
```

**2、编辑ld.so.conf文件**

```sh
echo '/usr/local/openssl-3.1.0/lib64' >> /etc/ld.so.conf
ldconfig -v
```

**3、创建操作系统软链接**

```sh
ln -s /usr/local/openssl-3.1.0/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl-3.1.0/include/openssl /usr/include/openssl
ll -s /usr/bin/openssl
ll -s /usr/include/openssl
```

**4、检查Openssl版本**

```sh
# openssl version
OpenSSL 3.1.0 14 Mar 2023 (Library: OpenSSL 3.1.0 14 Mar 2023)
```

### 3.5、编译安装openssh

**1、编译安装**

```sh
# cd openssh-9.3p1
# ./configure --prefix=/usr/local/openssh-9.3p1 --sysconfdir=/etc/ssh --with-pam \
--with-ssl-dir=/usr/local/openssl-3.1.0 --with-zlib=/usr/local/zlib-1.2.13 --without-hardening
# make && make install
```

**2、替换新版本openssh相关命令**

```sh
\cp -arf /usr/local/openssh-9.3p1/bin/scp /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/bin/sftp /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/bin/ssh /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/bin/ssh-add /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/bin/ssh-agent /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/bin/ssh-keygen /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/bin/ssh-keyscan /usr/bin/
\cp -arf /usr/local/openssh-9.3p1/sbin/sshd /usr/sbin/sshd
```

**3、拷贝启动脚本**

```sh
\cp -a contrib/redhat/sshd.init /etc/init.d/sshd
\cp -a contrib/redhat/sshd.pam /etc/pam.d/sshd.pam
chmod +x /etc/init.d/sshd
```

**5、修改配置文件**

```sh
vi /etc/ssh/sshd_config
PermitRootLogin yes
```

**6、设置开机启动,并验证版本**

```sh
systemctl daemon-reload
systemctl enable sshd.socket
systemctl start sshd
ssh -V
OpenSSH_9.3p1, OpenSSL 3.1.0 14 Mar 2023

注意：在configure openssh时，有设置参数 –with-pam，会提示PAM is enabled. You may need to install a PAM control file for sshd, otherwise password authentication may fail. Example PAM control files can be found in the contrib/subdirectory，在此之后登录服务器时，会发生登陆失败的问题。再次把UserPAM的参数禁用掉才可以正常登录服务器。
```

**解决方法如下：**

```sh
# 备份原文件/etc/pam.d/sshd，将以下内容覆盖写入/etc/pam.d/sshd
vim /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
-auth      optional     pam_reauthorize.so prepare
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
session    include      postlogin
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
```

## 六、版本回退

**回滚前：**

```sh
[root@localhost ~]# ssh -V
OpenSSH_9.3p1, OpenSSL 3.1.0 14 Mar 2023
```

**回滚操作：**

**1、回滚openssl**

```sh
rm -f /usr/bin/openssl 
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl 
rm -f /usr/include/openssl 
ln -s /usr/local/openssl/includeopenssl /usr/include/openssl
openssl version 
OpenSSL 1.1.1o  3 May 2022
```

**2、回滚openssh**

```sh
rm -rf /etc/ssh 
mv /etc/ssh_bak /etc/ssh 
\cp -arpf /usr/bin/bak/cp /usr/bin/cp
\cp -arpf /usr/bin/bak/sftp /usr/bin/sftp 
\cp -arpf /usr/bin/bak/ssh /usr/bin/ssh 
\cp -arpf /usr/bin/bak/ssh-add /usr/bin/ssh-add 
\cp -arpf /usr/bin/bak/ssh-agent /usr/bin/ssh-agent 
\cp -arpf /usr/bin/bak/ssh-keygen /usr/bin/ssh-keygen 
\cp -arpf /usr/bin/bak/ssh-keyscan /usr/bin/ssh-keyscan 
\cp -arpf /usr/sbin/sshd_bak /usr/sbin/sshd 
\cp -arpf /etc/sysconfig/sshd_bak /etc/sysconfig/sshd 
\cp -arpf /usr/lib/systemd/system/sshd.service_bak /usr/lib/systemd/system/sshd.service 
\cp -arpf /etc/pam.d/sshd.pam_bak /etc/pam.d/sshd.pam 
\cp -arpf /etc/pam.d/sshd_bak /etc/pam.d/sshd 
systemctl daemon-reload
systemctl restart sshd
rm -rf /usr/bin/bak
rm -f /usr/sbin/sshd_bak
rm -f /usr/lib/systemd/system/sshd.service_bak
rm -f /etc/pam.d/sshd.pam_bak
rm -f /etc/pam.d/sshd_bak
rm -f /etc/sysconfig/sshd_bak

```

**回滚后：**

```sh
[root@localhost ~]# ssh -V
OpenSSH_9.0p1, OpenSSL 1.1.1o  3 May 2022
```



































































































































































































































