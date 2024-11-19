 

最小化安装Ubuntu Server 22.04
------------------------

#### 0.Ubuntu Server下载

```java
https://ubuntu.com/download/server
```

### 1、自定义安装虚拟机

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f9f3e3ad44bd78391d87f993f831a9b.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2653f729fb928a1ad6a8550d8f70a29e.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c20f5a18d0df0cd3de92f5d1f54b60ca.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a66efd98ca21af51a5fb9b450a3c0ed1.png#pic_center)

**这里的cpu内存等硬件自定义设置，博主比较喜欢的配置是 2核2内核2G内存，后面其他的没有提及的都默认配置**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0833f6045f5fa863337181c3d687bd32.png#pic_center)

**这里最好使用NAT网络，方便后面远程连接**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03c63b007ae8bcc4a7ed6ff9e8dbe137.png#pic_center)

**完成配置后点击下面的CD，开始设置IOS路径**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22c8b9bf343ebbcb8a84c67b94211b45.png#pic_center)

##### **完成后确认，开机**

### 2、Ubuntu Server最小化安装

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a2f00caaa9e1a8fa361f24ff555a654.png#pic_center)

**选择第一个**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f76e5505250e46839add77697d46bbd4.png#pic_center)

**这里由于没有中文，只能选择英文**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6590a11ba431a4eae1896457fab6b0bf.png#pic_center)

**选择第二个，不更新**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/77ff7f16e33e516c84c6f1a9aa5a758b.png#pic_center)

**这里直接回车，默认配置**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a2389654b1b1ab7a7ff6303a10f33fb.png#pic_center)

**按上下键，到最小化安装那，按回车，等最小化安装前面括号有一个x时候，，下键到Done，回车**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe92635b478320a266a30334b7f86336.png#pic_center)

**这里上下键，到ens33那个下面，等到ip自动出来的时候，Done回车**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/872a487e6fd6dbedbe12d99c2f6172a1.png#pic_center)

**Configure proxy配置页面的Proxy address无需配置，直接回车**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/115b2ce7c132ce070b93a8943499fffc.png#pic_center)

**这里设置云源，自己输入以下阿里云的源**

阿里云源地址：`https://mirrors.aliyun.com/ubuntu`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f37c190f7a70305379e7ea963d154194.png#pic_center)

**选择安装磁盘，直接回车默认自动分配，需要手动分区的话选择 \[custom storage layout\]**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3667a203774a04e967d4bb85ce1669af.png#pic_center)

**检查磁盘分区是否符合你的要求，回车继续**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1f35cf49def360082dda78334f949a24.png#pic_center)

**再次确认 Continue 继续**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ad79b10d6b1428c7316cf1e15d271a5.png#pic_center)

**设置计算机名、用户名及密码**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e29eba432fd265dda6c5229cc3d6d8a3.png#pic_center)

**按空格键或者回车 选择安装 OpenSSH Server 服务**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c78bc6c68d3760009f23f758b85a2a6.png#pic_center)

**选择预置环境，按需选取，自由而定，不需要则直接选择Done回车继续**,

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/faa0e3b4e2ea7e3d0f9fac188f186f21.png#pic_center)

**开始安装，这个环节有点慢，需要有点耐心,下面那个小横再转就说明在安装，博主也安装了十几分钟**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d23c58500518d934cd85d6a79751538.png#pic_center)

**这里可以取消更新，直接重启，我们可以后续再更新，也可以等他更新完，博主建议等他更行完**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ef77f5d3e50bcb9571118c9fa743e5f6.png#pic_center)

**重启重启过程中可能有的地方会卡一下，你按一下回车就好**

#### 3、服务器配置

##### （1）设置root密码

重启后，开启root用户的设置，设置root用户密码：`sudo passwd root`

需要先输入当前用户密码，再输入需要设置的root用户密码

##### （2）安装需要的工具

`sudo apt install -y vim zip net-tools`

`sudo apt-get install inetutils-*ping*`

##### （3）设置静态ip

①获取当前ip `ip addr`

②编辑网络文件：`sudo vim /etc/netplan/00-installer-config.yaml`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/64cf34f42307e333df097e72ab9fb2d7.png#pic_center)

```shell
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      addresses:
        - 192.168.10.133/24
      nameservers:
        addresses: [114.114.114.114, 8.8.8.8]
      routes:
        - to: default
          via: 192.168.10.2
  version: 2
```

③配置生效： `sudo netplan apply`

#### 更新：

```shell
sudo apt-get update

sudo apt-get dist-upgrade
```

##### 完成啦！！！好好完你的服务器吧！

