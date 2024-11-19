# centos7离线安装图形界面

**CentOS** 系统如果需要图形界面，我们可以在安装系统的时候一起安装 **gnome**。但如果一开始没有安装也没关系，执行 **yum groupinstall -y "GNOME Desktop"** 命令即可自动下载安装。

  但在一些特殊情况时（比如纯内网环境下），如果想对目前已有的系统安装图像界面，我们是无法直接联网在线安装的，那么只能通过手动的方式进行安装，下面介绍具体的操作步骤。

1、**准备系统镜像**

```sh
（1）首先查看服务器上系统的版本，下载相应的系统镜像文件（下载地址）：
https://mirrors.aliyun.com/centos/
```

![原文:CentOS - 离线安装GNOME图形化界面教程](https://www.hangge.com/blog_uploads/202207/2022071114275463534.png)

```sh
（2）注意下载的是 Everything 版本，该版本里面包含完整安装版的内容，避免我们安装时缺少一些软件包：
```

![原文:CentOS - 离线安装GNOME图形化界面教程](https://www.hangge.com/blog_uploads/202207/2022071114300128070.png)

```sh
（3）将下载下来的镜像上传到服务器，并进行挂载：
mkdir /mnt/iso
]# mount /opt/iso/rhel-server-6.4-x86_64-dvd.iso /mnt -o loop 挂载镜像到该目录下

（4）挂载后执行如下命令，确认是否挂载成功：
df -h
```

![原文:CentOS - 离线安装GNOME图形化界面教程](https://www.hangge.com/blog_uploads/202207/2022070816534699321.png)

2、**修改系统的yum源**

```sh
（1）首先执行如下命令备份原 repo 文件：
[root@CentOS6 6]# cd /etc/yum.repos.d/
[root@CentOS6 yum.repos.d]# rm -rf * //删除下面所有的文件
[root@CentOS6 yum.repos.d]# vim my.repo //自定义名称 ，必须以.repo结尾

（2）然后新建一个 repo 文件：
vi local.repo

（3）文件内容如下，这样后续执行 yum 命令时都是从本地源进行安装：
[LocalRope]
name=local
enabled=1
gpgcheck=0
baseurl=file:///mnt/iso
```

3、**安装GNOME**

```sh
（1）我们执行如下命令安装图形界面软件 GNOME：
yum groupinstall -y "GNOME Desktop"

[root@CentOS6 ~]#yum grouplist #显示所有已安装和可用的套件
[root@CentOS6 ~]#yum groupinstall ‘X Window System’ #安装桌面环境
[root@CentOS6 ~]#yum groupinstall ‘Desktop’ #安装gnome

（2）安装完毕后即可启动图形界面：
init  5
```

4、**安装vncserver**

```sh
[root@rac1 ~]# yum install tigervnc-server -y
[root@rac1 ~]#vncpasswd
[root@rac1 sysconfig]# cat /etc/sysconfig/vncservers
VNCSERVERS=“1:root”
VNCSERVERARGS[2]="-geometry 1200x800"
[root@rac1 .vnc]# service vncserver restart
```

+ 备注：
+ **开机以命令模式启动，执行：**
  systemctl set-default multi-user.target
  **开机以图形界面启动，执行：**
  systemctl set-default graphical.target

5、**还原系统的yum源**

```sh
安装完毕后如果需要还原系统 yum 源，则可执行如下命令：
cd /etc/yum.repos.d
mv ./bak/* ./
rm -rf local.repo
rm -rf bak
```



































































































