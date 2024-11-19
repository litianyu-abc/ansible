 

### 前言

环境：`centos7.9 kernel版本：3.10.0-1160.71.1.el7.x86_64`  
kernel内核的官网：`https://www.kernel.org/`  
一个linux社区的仓库源，ELRepo官网：`http://elrepo.org/tiki/tiki-index.php`

### 说明

由于需要安装jumpserver，而jumpserver官网显示需要4.0以上的内核，所以需要升级内核：

```bash
#查看内核版本
[root@jump ~]# uname -r
3.10.0-1160.71.1.el7.x86_64
#有很多内核的包
[root@jump ~]# rpm -qa | grep kernel
kernel-3.10.0-1160.el7.x86_64
kernel-3.10.0-1160.71.1.el7.x86_64
kernel-devel-3.10.0-1160.71.1.el7.x86_64
kernel-headers-3.10.0-1160.71.1.el7.x86_64
kernel-tools-libs-3.10.0-1160.71.1.el7.x86_64
kernel-tools-3.10.0-1160.71.1.el7.x86_64
```

### 联网升级内核版本

1、方法一、下载官网的稳定版本内核： `wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.14.tar.xz`进行编译安装，由于内核编译安装不像平时的 `./confgue&& make && make install` 那么简单，所以本次不讲这种编译安装方式。

2、方法二，使用yum 更新内核：  
大多数 Linux 发行版提供自行维护的内核，可以通过 yum 或 rpm 等包管理系统升级。  
ELRepo是一个为Linux提供驱动程序和内核映像的存储库，本篇内核升级方案就是采用ELRepo提供的内核通道。  
ELRepo官网：`http://elrepo.org/tiki/tiki-index.php`

```bash
#导入公钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
#下载并安装elrepo仓库
yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
#官网提示安装的，那就安装
yum install -y yum-plugin-fastestmirror
#这里能离线下载到内核的rpm包,如果你的linux不联网,则可以离线下载安装包
https://elrepo.org/linux/kernel/el7/x86_64/RPMS/

#查看elrepo源里可用内核版本，可以看到有长期支持的版本5.4.257和稳定的主线版本6.0.0
# lt = long time     长期支持内核，采用长期支持版本（kernel-lt），更加稳定一些
# ml=mainline        稳定主线内核
[root@jump ~]# yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
Available Packages
kernel-lt.x86_64                      5.4.257-1.el7.elrepo        elrepo-kernel
kernel-lt-devel.x86_64                5.4.257-1.el7.elrepo        elrepo-kernel
kernel-lt-doc.noarch                  5.4.257-1.el7.elrepo        elrepo-kernel
kernel-lt-headers.x86_64              5.4.257-1.el7.elrepo        elrepo-kernel
kernel-lt-tools.x86_64                5.4.257-1.el7.elrepo        elrepo-kernel
kernel-lt-tools-libs.x86_64           5.4.257-1.el7.elrepo        elrepo-kernel
kernel-lt-tools-libs-devel.x86_64     5.4.257-1.el7.elrepo        elrepo-kernel
kernel-ml.x86_64                      6.0.0-1.el7.elrepo          elrepo-kernel
kernel-ml-devel.x86_64                6.0.0-1.el7.elrepo          elrepo-kernel
kernel-ml-doc.noarch                  6.0.0-1.el7.elrepo          elrepo-kernel
kernel-ml-headers.x86_64              6.0.0-1.el7.elrepo          elrepo-kernel
kernel-ml-tools.x86_64                6.0.0-1.el7.elrepo          elrepo-kernel
kernel-ml-tools-libs.x86_64           6.0.0-1.el7.elrepo          elrepo-kernel
kernel-ml-tools-libs-devel.x86_64     6.0.0-1.el7.elrepo          elrepo-kernel
perf.x86_64                           5.19.12-1.el7.elrepo        elrepo-kernel
python-perf.x86_64                    5.19.12-1.el7.elrepo        elrepo-kernel

#安装长期稳定版本，稳定可靠，版本为5.4.257
yum --enablerepo=elrepo-kernel install kernel-lt-5.4.257-1.el7.elrepo.x86_64
#安装5.4.257版本的这些依赖包
#安装之前需要卸载低版本内核的kernel-tools和kernel-headers，不然会报冲突
yum remove -y kernel-tools* kernel-headers*
yum --enablerepo=elrepo-kernel install kernel-lt-devel-5.4.257-1.el7.elrepo.x86_64 \
kernel-lt-doc-5.4.257-1.el7.elrepo.noarch \
kernel-lt-headers-5.4.257-1.el7.elrepo.x86_64 \
kernel-lt-tools-5.4.257-1.el7.elrepo.x86_64 \
kernel-lt-tools-libs-5.4.257-1.el7.elrepo.x86_64 \
kernel-lt-tools-libs-devel-5.4.257-1.el7.elrepo.x86_64 -y
 
#查看内核是否载入到grub2
#/etc/grub2.cfg文件是/boot/grub2/grub.cfg文件的软链接，grub.cfg文件是grub引导加载器的配置文件
#注意：如果服务器使用efi模式启动，那么可能没有/boot/grub2/grub.cfg文件，其对应的配置文件可能是/boot/efi/EFI/centos/grub.cfg
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.4.257-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.71.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-20220727115107055496013495205931) 7 (Core)

刚刚安装的内核即0 : CentOS Linux (5.4.257-1.el7.elrepo.x86_64) 7 (Core)
我们需要把grub默认设置为0
可以通过 grub2-set-default 0 命令设置或编辑 vim /etc/default/grub 文件来设置

vim /etc/default/grub
GRUB_DEFAULT=0		#这句改为GRUB_DEFAULT=0 意思是GRUB初始化页面的第一个内核将作为默认内核，保存退出

#检查默认内核是不是5.4
grubby --default-kernel
#生成 grub 配置文件
grub2-mkconfig -o /boot/grub2/grub.cfg
#重启linux系统
reboot

#重启完成，查看现在linux内核，已经变成5.4.257了
[root@jump ~]# uname -r
5.4.257-1.el7.elrepo.x86_64
[root@jump ~]# 

#查看系统中的全部内核，现在旧的3.10内核还在
[root@jump ~]# rpm -qa | grep kernel
kernel-lt-tools-5.4.257-1.el7.elrepo.x86_64
kernel-lt-devel-5.4.257-1.el7.elrepo.x86_64
kernel-3.10.0-1160.el7.x86_64
kernel-lt-tools-libs-5.4.257-1.el7.elrepo.x86_64
kernel-lt-tools-libs-devel-5.4.257-1.el7.elrepo.x86_64
kernel-lt-headers-5.4.257-1.el7.elrepo.x86_64
kernel-3.10.0-1160.92.1.el7.x86_64
kernel-lt-5.4.257-1.el7.elrepo.x86_64
kernel-lt-doc-5.4.257-1.el7.elrepo.noarch

#可以选择移除全部旧内核版本的相关包（自行决定是否卸载）
#补充一句，Linux是支持多版本内核共存的，无非是系统启动的时候应用哪个版本内核而已
yum remove kernel-3.10.0-1160* -y
#查看grub2已经没有3.10的内核了
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.4.257-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (0-rescue-bfd5b36eb1f547e18ff398ed50be7142) 7 (Core)

#发现卸载3.10.0-1160版本内核的时候把这些依赖给卸载了，所以现在重新安装这些依赖
yum install  gcc   gcc-c++ glibc-devel  glibc-headers -y

#查看现在的内核，都是5.4.257-1版本的了
[root@jump ~]# rpm -qa | grep kern
kernel-lt-tools-5.4.257-1.el7.elrepo.x86_64
kernel-lt-devel-5.4.257-1.el7.elrepo.x86_64
kernel-lt-tools-libs-5.4.257-1.el7.elrepo.x86_64
kernel-lt-tools-libs-devel-5.4.257-1.el7.elrepo.x86_64
kernel-lt-headers-5.4.257-1.el7.elrepo.x86_64
kernel-lt-5.4.257-1.el7.elrepo.x86_64
kernel-lt-doc-5.4.257-1.el7.elrepo.noarch
#再次重启linux（可不用重启）
init 6

一切正常，Linux内核升级完成。
```

### 离线升级内核

```bash
从官网可以下载内核rpm包：https://elrepo.org/linux/kernel/el7/x86_64/RPMS/
#找一台可以联网的服务器下载内核包
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-5.4.257-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-devel-5.4.257-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-doc-5.4.257-1.el7.elrepo.noarch.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-headers-5.4.257-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-tools-5.4.257-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-tools-libs-5.4.257-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-tools-libs-devel-5.4.257-1.el7.elrepo.x86_64.rpm
#将下载好的rpm上传到内网服务器
[root@jump update-kernel]# ll
-rw-r--r--. 1 root root 52921216 Sep 24 01:46 kernel-lt-5.4.257-1.el7.elrepo.x86_64.rpm
-rw-r--r--. 1 root root 13538544 Sep 24 01:46 kernel-lt-devel-5.4.257-1.el7.elrepo.x86_64.rpm
-rw-r--r--. 1 root root  8625312 Sep 24 01:46 kernel-lt-doc-5.4.257-1.el7.elrepo.noarch.rpm
-rw-r--r--. 1 root root  1399908 Sep 24 01:46 kernel-lt-headers-5.4.257-1.el7.elrepo.x86_64.rpm
-rw-r--r--. 1 root root   232316 Sep 24 01:46 kernel-lt-tools-5.4.257-1.el7.elrepo.x86_64.rpm
-rw-r--r--. 1 root root   118976 Sep 24 01:46 kernel-lt-tools-libs-5.4.257-1.el7.elrepo.x86_64.rpm
-rw-r--r--. 1 root root    96432 Sep 24 01:46 kernel-lt-tools-libs-devel-5.4.257-1.el7.elrepo.x86_64.rpm
#安装内核
yum install -y ./kernel-lt-5.4.257-1.el7.elrepo.x86_64.rpm
yum install -y ./kernel-lt-devel-5.4.257-1.el7.elrepo.x86_64.rpm
yum install -y ./kernel-lt-doc-5.4.257-1.el7.elrepo.noarch.rpm
yum install -y ./kernel-lt-headers-5.4.257-1.el7.elrepo.x86_64.rpm
yum install -y ./kernel-lt-tools-libs-5.4.257-1.el7.elrepo.x86_64.rpm
yum install -y ./kernel-lt-tools-5.4.257-1.el7.elrepo.x86_64.rpm
yum install -y ./kernel-lt-tools-libs-devel-5.4.257-1.el7.elrepo.x86_64.rpm
#查看内核是否载入到grub2
#/etc/grub2.cfg文件是/boot/grub2/grub.cfg文件的软链接，grub.cfg文件是grub引导加载器的配置文件
#注意：如果服务器使用efi模式启动，那么可能没有/boot/grub2/grub.cfg文件，其对应的配置文件可能是/boot/efi/EFI/centos/grub.cfg
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.4.257-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.92.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-bfd5b36eb1f547e18ff398ed50be7142) 7 (Core)

刚刚安装的内核即0 : CentOS Linux (5.4.257-1.el7.elrepo.x86_64) 7 (Core)
我们需要把grub默认设置为0
可以通过 grub2-set-default 0 命令设置或编辑 vim /etc/default/grub 文件来设置

vim /etc/default/grub
GRUB_DEFAULT=0		#这句改为GRUB_DEFAULT=0 意思是GRUB初始化页面的第一个内核将作为默认内核，保存退出

#检查默认内核是不是5.4
grubby --default-kernel
#生成 grub 配置文件
grub2-mkconfig -o /boot/grub2/grub.cfg
#重启linux系统
reboot
#重启完成，查看现在linux内核，已经变成5.4.257了
uname -r
#安装完成之后，会在/usr/src/kernels目录生成内核的源码包，这个仅了解即可
[root@jump ~]# ls /usr/src/kernels/5.4.257-1.el7.elrepo.x86_64/
arch  block  certs  crypto  drivers  fs  include  init  ipc  Kconfig  kernel  lib  Makefile  mm  Module.symvers  net  samples  scripts  security  sound  System.map  tools  usr  virt
```

本文转自 <https://blog.csdn.net/MssGuo/article/details/127184206>，如有侵权，请联系删除。