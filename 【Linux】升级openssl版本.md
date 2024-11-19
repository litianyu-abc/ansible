 

**目录**

[摘要](#t0)

[准备工作](#t1)

[1.查看openssl的版本](#t2)

[2.查看openssl的路径](#t3)

[3.备份openssl文件](#t4)

[4.下载openssl](#t5)

[升级openssl](#t6)

[1.解压openssl](#t7)

[2.切换到解压好的openssl目录](#t8)

[3.配置openssl安装目录](#t9)

[4.编译&&安装](#t10)

[5.创建软链接](#t11)

[6.添加动态链接库数据](#t12)

[7.更新动态链接库：ldconfig -v](#t13)

[8.验证openssl](#t14)

* * *

### 摘要

为什么要升级openssl版本，一是解决旧的OpenSSL版本可能会存在一些安全漏洞。这些漏洞可能会被黑客利用，对系统和数据造成威胁，因此，升级到新的OpenSSL版本可以修复这些已知的安全漏洞，提高系统的安全性。二是有些软件的安装部署会涉及到openssl的版本，如果版本不满足，就需要升级openssl或降级，比如：Nginx、RabbitMQ等安装的时候会涉及到openssl。

### 准备工作

#### 1.查看openssl的版本

openssl version

![](https://img-blog.csdnimg.cn/direct/61fad190cc9c4e199ee314a40b3ce8bc.png)

可以看成目前openssl的版本是1.1.1f版本还是较旧。

#### 2.查看openssl的路径

![](https://img-blog.csdnimg.cn/direct/3919212b5395484c8c61c3e1f8e40e55.png)

#### 3.备份openssl文件

mv /usr/bin/openssl /usr/bin/openssl\_old

mv /usr/include/openssl /usr/include/openssl\_old

![](https://img-blog.csdnimg.cn/direct/c6cc929faa20458f9afe7a926bfe4c98.png)

#### 4.下载openssl

官方地址：

[\[ Downloads \] - /source/index.html](https://www.openssl.org/source/ "[ Downloads ] - /source/index.html")

![](https://img-blog.csdnimg.cn/direct/f1352fde32ba4509b45bee929180e7d9.png)

目前在做替换的时候，选择下载的是openssl-3.0.13版本

方式一：点击openssl-3.0.13版本即可下载

![](https://img-blog.csdnimg.cn/direct/20d433f5883547a9bf76038454a17ad0.png)

方式二：如果服务器是联网的状态可以通过命令下载

```cobol
wget --no-check-certificate https://www.openssl.org/source/openssl-3.0.13.tar.gz
```

 参数：`--no-check-certificate`  会跳过证书验证  
下载好之后，上传到服务器的某个目录，在此上传的目录是/opt/

### 升级openssl

#### 1.解压openssl

```cobol
tar -zxvf openssl-3.0.13.tar.gz
```

![](https://img-blog.csdnimg.cn/direct/81b56278b788424082ce968f5a5abc00.png)

#### 2.切换到解压好的openssl目录

```cobol
cd openssl-3.0.13/
```

#### 3.配置openssl安装目录

```cobol
./config --prefix=/usr/local/openssl
```

![](https://img-blog.csdnimg.cn/direct/e2433db173f54beeafd8e5e553595a6a.png)

#### 4.编译&&安装

```go
make && make install
```

等待安装完成即可。

#### 5.创建软链接

说明：创建的软链接和之前没升级通过whereis openssl保持一致即可。

```cobol
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl 
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
```

![](https://img-blog.csdnimg.cn/direct/f82f4d84aa05477cae45f0d46370decd.png)

#### 6.添加动态链接库数据

echo "/usr/local/openssl/lib/" >> /etc/ld.so.conf

![](https://img-blog.csdnimg.cn/direct/1e9fc121b9a641d4822b7f5490a23ce3.png)

检查一下，已经在/etc/ld.so.conf中存在。

 ![](https://img-blog.csdnimg.cn/direct/9159b0fd62054565bec21dba7f9d007a.png)

#### 7.更新动态链接库：ldconfig -v

#### 8.验证openssl

查看openssl版本 openssl version -a会显示全面详细信息。

![](https://img-blog.csdnimg.cn/direct/fbdd02c1ed55487d98e29f220e945af1.png)

到此openssl升级完成。







# CentOS7编译安装OpenSSL3.1

## 1.下载Openssl源码包

官网：https://www.openssl.org/

```shell
[root@localhost ~]# wget https://www.openssl.org/source/openssl-3.1.0.tar.gz
```

## 2.解压安装

```shell
[root@localhost ~]# tar -xvf openssl-3.1.0.tar.gz -C /usr/local/
[root@localhost ~]# cd /usr/local/openssl-3.1.0/
[root@localhost openssl-3.1.0]# ./config --prefix=/usr/local/openssl
```

如果报错为: 缺少IPC/Cmd.pm模块

```shell
[root@localhost openssl-3.1.0]# ./config  --prefix=/usr/local/openssl
Can't locate IPC/Cmd.pm in @INC (@INC contains: /root/Downloads/openssl-3.1.0/util/perl /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 . /root/Downloads/openssl-3.1.0/external/perl/Text-Template-1.56/lib) at /root/Downloads/openssl-3.1.0/util/perl/OpenSSL/config.pm line 18.
BEGIN failed--compilation aborted at /root/Downloads/openssl-3.1.0/util/perl/OpenSSL/config.pm line 18.
Compilation failed in require at /root/Downloads/openssl-3.1.0/Configure line 23.
BEGIN failed--compilation aborted at /root/Downloads/openssl-3.1.0/Configure line 23.
```

解决方法：

```shell
[root@localhost openssl-3.1.0]# yum -y install perl-IPC-Cmd
```

## 3.重新编译配置：

```shell
[root@localhost openssl-3.1.0]# ./config  --prefix=/usr/local/openssl
[root@localhost openssl-3.1.0]# make -j 32
[root@localhost openssl-3.1.0]# make install
```

libssl.so.3文件在/usr/local/openssl/lib64目录下面，需要配置到共享库中：

![img](https://img2023.cnblogs.com/blog/1513031/202305/1513031-20230511170347476-2131932986.png)

```shell
[root@localhost ~]# vim /etc/ld.so.conf
include ld.so.conf.d/*.conf
/usr/local/openssl/lib64

加载生效：
[root@localhost ~]# ldconfig
```

做软连接：

```shell
[root@localhost ~]# mv /usr/bin/openssl /usr/bin/openssl.bak
[root@localhost ~]# mv /usr/include/openssl /usr/include/openssl.bak 

[root@localhost ~]# ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
[root@localhost ~]# ln -s /usr/local/openssl/include/openssl/  /usr/include/openssl
```

查看Openssl版本：

```shell
[root@localhost ~]# openssl version
OpenSSL 3.1.0 14 Mar 2023 (Library: OpenSSL 3.1.0 14 Mar 2023)
```

