 

**问题**：插入网线后，无法识别有线连接。  
**原因**：主板有线网卡的型号和系统中网卡的驱动不匹配

### 解决

**前提条件**： 需要连wifi通网，因为需要安装dkms，和下载驱动。

#### 删除原驱动

1、查看网卡型号

```
lspci | grep net
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f27cd8bb7ab279a1fe5196b6b6c415dc.png)  
我的网卡型号是8125

2、查看网卡驱动

```
lspci -k
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b7eb7a4e270434393b5f15f571ff355.png)  
网卡驱动是r8169

3、卸载网卡驱动  
找出位置

```
modinfo r8169 | grep filename
```

然后删除

```
rm path/r8169.ko
```

卸载驱动

```
rmmod r8169
```

查看是否卸载成功

```
lsmod | grep r8169
```

#### 安装驱动

1、在[官网](https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software)找到相应的驱动，这里我找的是`driver r8125`的`9.005.06`版本。下载好后，解压：

```
sudo tar xvf r8125-9.005.06.tar.bz2  -C /usr/src
```

2、在`/usr/src/r8125-9.005.06`中创建`dkms.conf`，内容为：

```
PACKAGE_NAME=Realtek_r8125
PACKAGE_VERSION=9.005.06

DEST_MODULE_LOCATION=/updates/dkms
BUILT_MODULE_NAME=r8125
BUILT_MODULE_LOCATION=src/

MAKE="'make' -C src/ all"
CLEAN="'make' -C src/ clean"
AUTOINSTALL="yes"
```

3、安装dkms

```
sudo apt-get install dkms
```

4、编译DKMS并且挂载驱动

```
sudo dkms add -m r8125 -v 9.005.06
sudo dkms build -m r8125 -v 9.005.06
sudo dkms install -m r8125 -v 9.005.06
sudo depmod -a
sudo modprobe r8125
```

成功后，有线图标会出现  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb65a5e372d3f6df33c1d5498d3536ca.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b44333747915c8e671122ebed5a950d.png)

本文转自 <https://blog.csdn.net/weixin_43932656/article/details/118007962>，如有侵权，请联系删除。