# centos7离线安装ntp时间服务器

## 1.检测ntp是否安装

（1）【命令】rpm –qa | grep ntp

![img](https://upload-images.jianshu.io/upload_images/11847494-e0ee15b71df26963.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

（2）如果未安装执行命令

###  1.2.1在线安装

 【命令】yum –y install ntp

####  1.2.2完全离线安装

 因本地centos完全离线，下载rpm到本地安装
 本地centos版本为：7.5.1804
 因系统自带ntpdate所以无需下载，可以先查看本地自带版本

![img](https://upload-images.jianshu.io/upload_images/11847494-64cc151a82c8e421.png?imageMogr2/auto-orient/strip|imageView2/2/w/622/format/webp)

##### 1.2.2.1下载对应版本ntp及相应依赖的rpm

 ntp-4.2.6p5-28.el7.centos.x86_64.rpm
 autogen-libopts-5.18-5.el7.x86_64.rpm
 下载地址
 ntp下载地址
 [http://www.rpmfind.net/linux/rpm2html/search.php?query=ntp&submit=Search+...&system=&arch=](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.rpmfind.net%2Flinux%2Frpm2html%2Fsearch.php%3Fquery%3Dntp%26submit%3DSearch%2B...%26system%3D%26arch%3D)

![img](https://upload-images.jianshu.io/upload_images/11847494-0a840c89cd0dec02.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

autogen-libopts 下载地址
 [http://www.rpmfind.net/linux/rpm2html/search.php?query=autogen-libopts&submit=Search+...&system=&arch=](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.rpmfind.net%2Flinux%2Frpm2html%2Fsearch.php%3Fquery%3Dautogen-libopts%26submit%3DSearch%2B...%26system%3D%26arch%3D)

![img](https://upload-images.jianshu.io/upload_images/11847494-5bee5bc975b8eb52.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

###### 1.2.2.2使用SecureCRT上传rpm包

登陆要上传的linux服务器

![img](https://upload-images.jianshu.io/upload_images/11847494-3a2baa4b012db686.png?imageMogr2/auto-orient/strip|imageView2/2/w/688/format/webp)

打开sftp界面，使用put命令上传rpm文件

![img](https://upload-images.jianshu.io/upload_images/11847494-c4c837634f32d2be.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

上传完成后到存放目录下可以查看

![img](https://upload-images.jianshu.io/upload_images/11847494-65bb884d6483c873.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

###### 1.2.2.3执行一下命令进行安装

 rpm -ivh autogen-libopts-5.18-5.el7.x86_64.rpm
 rpm -ivh ntp-4.2.6p5-28.el7.centos.x86_64.rpm
 【命令】rpm -qa | grep ntp 查看是否安装完成

![img](https://upload-images.jianshu.io/upload_images/11847494-d8095f40ab5973ea.png?imageMogr2/auto-orient/strip|imageView2/2/w/504/format/webp)

###### 1.2.2.4 安装完成后设置ntp开机启动并启动ntp，如下：

systemctl enable ntpd

systemctl start ntpd

###### 1.2.2.5 查看ntp服务器状态

【命令】service ntpd status

![img](https://upload-images.jianshu.io/upload_images/11847494-e459fd1fe83428c2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 2.配置服务器时间源

【命令】vim /etc/ntp.conf
在server下面配置时间源服务器

![img](https://upload-images.jianshu.io/upload_images/11847494-9a00cf7b71d7d8ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 3.启动ntp，查看状态

（1）查看ntp状态，启动ntp
【命令】service ntpd status

![img](https://upload-images.jianshu.io/upload_images/11847494-838410af8445873c.png?imageMogr2/auto-orient/strip|imageView2/2/w/878/format/webp)

【命令】service ntpd start

![img](https://upload-images.jianshu.io/upload_images/11847494-6d33f2447b3ffae7.png?imageMogr2/auto-orient/strip|imageView2/2/w/932/format/webp)

（2）查看ntp服务器有无和上层ntp连通
【命令】ntpstat

刚配置完成，查询是失败的，时间未同步，需要一定的时间进行同步

![img](https://upload-images.jianshu.io/upload_images/11847494-46e940f53794f3bf.png?imageMogr2/auto-orient/strip|imageView2/2/w/502/format/webp)

同步成功

![img](https://upload-images.jianshu.io/upload_images/11847494-1082d0decc563a9d.png?imageMogr2/auto-orient/strip|imageView2/2/w/836/format/webp)

（3）查看ntp服务器与上层ntp的状态
【命令】ntpq -p

![img](https://upload-images.jianshu.io/upload_images/11847494-2a06c463ff8bbbf1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1052/format/webp)































































































































