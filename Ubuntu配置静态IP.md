 

#### 文章目录

*   [方法一](#_4)
*   *   [第一步：查看当前主机网络信息](#_6)
    *   [第二步：修改配置文件](#_22)
    *   [第三步：使配置生效且检查网络连接状况](#_58)
*   [方法二](#_66)
*   *   [第一步：查看当前主机网络信息](#_68)
    *   [第二步：修改配置文件](#_81)
    *   [第三步：使配置生效且检查网络连接状况](#_106)
*   [结束](#_116)

* * *

方法一
---

**以`Ubuntu20.04`示例**

### 第一步：查看当前主机网络信息

```shell
ifconfig
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4f14f168d240d81301d01f34ab9e0bc.png)

本机网卡名为：`ens32`，`IP`地址为：`192.168.15.133`，[子网掩码](https://so.csdn.net/so/search?q=%E5%AD%90%E7%BD%91%E6%8E%A9%E7%A0%81&spm=1001.2101.3001.7020)为：`255.255.255.0`  
  

```shell
route -n
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f863e0004601072d7489398653a22e83.png)

网关地址为：`192.168.15.2`

基本信息查到，可以进行配置文件的修改了。

### 第二步：修改配置文件

*   进入配置文件夹

```shell
cd /etc/netplan
```

可以把配置文件复制一个备份

```shell
sudo cp 01-network-manager-all.yaml 01-network-manager-all.yaml-before
```

我这里就不复制了，直接编辑。  
编辑配置文件

```shell
sudo vim 01-network-manager-all.yaml
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/392a420e5f9e70feff815fdf5c879d4c.png)  
配置成如下图片即可：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f76fdb815653330277666de23a5ace6b.png)  
注意格式要正确，每个冒号后要留一个空格，这里提供可复制版本，按照自己的稍微改一下即可！

```
# Let NetworkManager manage all devices on this system
network:
  ethernets:
    ens32:
      addresses: [192.168.15.132/24]          # 设置静态IP地址和掩码
      routes:                                 # 设置网关地址
       - to: default
         via: 192.168.15.2
      dhcp4: false                            # 禁用dhcp
      nameservers:
        addresses: [114.114.114.114, 8.8.8.8] # 设置主、备DNS
  version: 2
  renderer: NetworkManager
```

**2024年补充**  
之前设置不知道为啥不行，现补一个新的：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/887c9408063492a18ebb8b91a3cbfc1a.png)

### 第三步：使配置生效且检查网络连接状况

配置完成后要执行命令`sudo netplan apply`使配置生效。  
执行`ifconfig`命令查看当前[IP地址](https://so.csdn.net/so/search?q=IP%E5%9C%B0%E5%9D%80&spm=1001.2101.3001.7020)，发现已经变更。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c00b4f7b16c04fb370489c907353a4bb.png)  
`ping`一下百度，发现网络可用。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c594382b1ff022e77544adb8bd9c4bf.png)  
在这里第一个方法设置静态IP成功，有的时候不知道配置了啥，在`Ubuntu`中找不到`netplan`文件夹，这个时候可以使用第二个方法。

方法二
---

**以`Ubuntu16.04`示例**

### 第一步：查看当前主机网络信息

```shell
ifconfig
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0160a7d702c0ec0e28a75cb2f19cc502.png)  
本机网卡名为：`ens32`，`IP`地址为：`192.168.15.134`，子网掩码为：`255.255.255.0`  
  

```shell
route -n
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9348352f6558e5ea74800a72f97102b3.png)  
网关地址为：`192.168.15.2`

### 第二步：修改配置文件

```shell
cd /etc/network
ls
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/120e33b183d8956f6102959ecd2d0e65.png)  
编辑`interfaces`文件

```shell
sudo vim interfaces
```

编辑成如下图片样子，即可设置成静态IP  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a09d5a550d1741bf2e8a83eee876d4c8.png)  
同样需要注意格式

```
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback
   
auto ens32
iface ens32 inet static
address 192.168.15.134
netmask 255.255.255.0
gateway 192.168.15.2
dns-nameservers 114.114.114.114
```

### 第三步：使配置生效且检查网络连接状况

重启一下使配置生效

```shell
sudo reboot
```

`ping`一下检查网络连接状态

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7616859e34795a26a16f771d39206465.png)

结束
--

OK！结束！

[参考文献](https://blog.csdn.net/fun_tion/article/details/126750615)

本文转自 <https://blog.csdn.net/weixin_58305495/article/details/130554393>，如有侵权，请联系删除。