## CentOS7 OpenSSL升级1.1.1w；OpenSSH 升级 9.5p1 

#### 一、 前言

OpenSSH 的加密功能需要用到OpenSSL，所以在升级OpenSSH的时候，大部分情况是需要将OpenSSL一起升级的。

这里我们选择先升级OpenSSL到OpenSSL 1.1.1w 11 Sep 2023

然后再升级OpenSSH 到OpenSSH_9.5p1, OpenSSL 1.1.1w 11 Sep 2023

两个都是最新版本，方便大家食用

##### 1.1 注意点

在升级之前先将需要的包上传到服务器，以免升级失败后导致上传文件失败。（这里我用的sftp上传，sftp的核心也需要用到SSH），并安装telnet-server服务保证SSH升级失败后，可以继续远程连接。

需要有初始yum源，要不然安装这两个服务众多的依赖包将会是梦魇。

原始版本信息如下

![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155548115-1928211354.png)

#### 二、升级OpenSSL

##### 2.1 安装依赖

```bash
yum -y install gcc*
 
```

##### 2.2备份、卸载原有OpenSSL

1、 查找openssl 相关目录，然后备份

```bash
[root@vm206 etc]# whereis openssl

openssl: /usr/bin/openssl /usr/lib64/openssl /usr/share/man/man1/openssl.1ssl.gz

[root@vm206 etc]# mv /usr/bin/openssl  /usr/bin/openssl.old

[root@vm206 etc]# mv /usr/lib64/openssl /usr/lib64/openssl.old
```

2、 卸载 openssl (这一步看个人需要，我有洁癖所以我卸载了)

```bash
yum remove openssl
```

##### 2.3安装openssl

```bash
tar -xzvf openssl-1.1.1w.tar.gz

cd openssl-1.1.1w/

./config --prefix=/usr/local/openssl

make && make install
```

*#这里我的目录选择了/usr 是因为系统最初始的openssl的目录就是/usr 这样可以省去的软连接、更新链接库的问题*

**创建软连接**

```sh
#我们要做的就是将 /usr/local/openssl/bin 目录下的 openssl 软件建立快捷方式放置到 /usr/bin目录下
# ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl

这时可能会提示 /usr/bin/openssl 已存在 或 /usr/bin/openssl is exist
这表示我们之前已经有了一个旧版本的openssl，我们将之删除即可。
#  rm -rf /usr/bin/openssl

注意：由于我之前已经采用了备份，采用mv备份openssl 为：openssl .old ，所以这里就没有提示了，如果你的出现提示则一定要删除，然后重新执行上面的命令即可创建完成
如果一切正常，这是就可以通过 openssl version 命令查看当前版本了

```

最后一步，彻底解决版本号不一致问题
添加动态库的环境变量
首先使用vi 或 vim 打开 /etc/ld.so.conf 文件

```sh
vim  /etc/ld.so.conf
然后添加以下内容
/usr/local/openssl/lib
```

(注意：如果你安装的不是openssl-1.1.1w 版本，则目录是不同的，openssl-3.0.12 的目录则为：/usr/local/ssl/lib64 ，这是让人开始安装openssl最容易搞不明白的地方。)
然后保存退出，应用配置。

##### 2.4验证![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155522791-543814966.png)

```bash
[root@vm206 openssl-1.1.1w]# whereis openssl

openssl: /usr/bin/openssl /usr/lib64/openssl /usr/include/openssl /usr/share/man/man1/openssl.1ssl.gz /usr/share/man/man1/openssl.1

[root@vm206 openssl-1.1.1w]# openssl  version

OpenSSL 1.1.1w  11 Sep 2023
```

*#可以看到我这边的目录和老版本的openssl的目录保持了一致，唯一不同的是多了一个/usr/include/openssl 库目录*

*#如果不加prefix ,openssl的默认路径如下*

```bash
Bin：  /usr/local/bin/openssl

include库  ：/usr/local/include/openssl

lib库：/usr/local/lib64/

engine库：/usr/lib64/openssl/engines
```

#### 三、 升级OpenSSH

##### 3.1 安装telnet-server

```bash
yum install telnet* -y

systemctl  start telnet.socket

systemctl enable telnet.socket

mv /etc/securetty /etc/securetty.bak
```

*#临时关闭安全登录，否则无法进行远程telnet连接*

*#有防火墙记得关闭防火墙，并关闭SELinux*

 ![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155638327-1856685937.png)

*#如上，已经可以通过telnet远程连接了，这下可以放心大胆的操作了。*

##### 3.2 安装依赖包

```bash
yum install -y gcc pam-devel rpm-build wget zlib-devel openssl-devel net-tools
```

##### 3.3 备份

 ![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155701185-1777270874.png)

*#通过whereis ssh sshd找出bin文件、源文件，然后备份。 man手册不需要备份。*

```bash
mv /etc/ssh /etc/ssh.bak

mv /usr/bin/ssh /usr/bin/ssh.bak

mv /usr/sbin/sshd /usr/sbin/sshd.bak

mv /etc/pam.d/sshd  /etc/pam.d/sshd.old
```

*#备份pam验证文件*

##### 3.4卸载旧版OpenSSH

```bash
yum remove openssh
```

##### 3.5安装新版OpenSSH

```bash
tar -xzvf openssh-9.5p1.tar.gz

cd openssh-9.5p1

./configure --prefix=/usr --sysconfdir=/etc/ssh  --with-pam   --with-ssl-dir=/usr/local/lib64/
```

*#其中--prefix --sysconfdir 这两个参数我仍然采用了系统之前的默认路径，避免路径混乱导致的问题*

```bash
make

make install

cd /etc/pam.d/

mv sshd.old sshd
```

*#恢复ssh pam认证*

```bash
cd openssh-9.5p1/

cp contrib/redhat/sshd.init /etc/init.d/sshd

chkconfig --add sshd

systemctl  enable sshd

systemctl  start sshd
```

 ![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155724986-1547662839.png)

*#可以看到，已经升级成功*

##### 3.6修改/etc/ssh/sshd_config 配置文件

​    文件修改如下，然后重启sshd服务即可

![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155800338-265052039.png)

![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155805068-1583318309.png)

![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155809734-1253756552.png)

​    登录成功界面

![img](https://img2023.cnblogs.com/blog/1461814/202310/1461814-20231027155830359-1214191357.png)

 





### Linux安装openssl出现Can‘t locate IPC/Cmd.pm in，error:1407742E:SSL routines:SSL23_GET_SERVER_HELLO:tlsv1

1. ##### 问题描述 Can‘t locate IPC/Cmd.pm in

  缺少IPC/Cmd.pm 模块

```sh
Can't locate IPC/Cmd.pm in @INC (@INC contains: /opt/common/openssl-3.0.1/util/perl /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 . /opt/common/openssl-3.0.1/external/perl/Text-Template-1.56/lib) at /opt/common/openssl-3.0.1/util/perl/OpenSSL/config.pm line 18.
BEGIN failed--compilation aborted at /opt/common/openssl-3.0.1/util/perl/OpenSSL/config.pm line 18.
Compilation failed in require at /opt/common/openssl-3.0.1/Configure line 23.
BEGIN failed--compilation aborted at /opt/common/openssl-3.0.1/Configure line 23.

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9a71f713e5684b53a578639c9a67d38a.png)

1. ##### 解决方案

  安装perl-CPAN

  ```sh
   yum install -y perl-CPAN
  ```

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/ab51c87cb3b34192843254574d1c17ab.png)

进入perl shell中

```sh
perl -MCPAN -e shell
```

进入后第一步选yes
第二步选manual
第三步选yes

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f10700bf6b44bd8baa70ffc6e147f7a.png)

出现以下cpan[1]>就可以了

![在这里插入图片描述](https://img-blog.csdnimg.cn/9c90507ee32c49a087e61c599a92f4f9.png)

安装缺少的模块

```sh
install IPC/Cmd.pm
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/103f2a36cfae44e3bb7c8e08468b430a.png)