 

1.  检查openssh版本：

升级前openssh 版本为7.4    openssl 版本为1.0.2k

![](https://img-blog.csdnimg.cn/direct/109aa9b82a7341fa9298f91144605e8d.png)

Openssh9.6   所需openssl >=1.1.1  因此openssl也需要升级。

1.  为了防止升级失败，无法使用SSH登录，首先安装telnet 预防。查看是否安装了telnet 客户端及服务

![](https://img-blog.csdnimg.cn/direct/222a92d3a74c4e8cbb4bcfb234a75469.png)

未安装telnet 服务及客户端。首先进行安装。

关闭防火墙

![](https://img-blog.csdnimg.cn/direct/81579d82025742a994416c5f4e5c3409.png)

进入opt/telnet目录中安装telnet，telnet服务需要xinetd-2.3.15-14.el7.x86\_64.rpm  首先安装。

![](https://img-blog.csdnimg.cn/direct/afd37099e9fd44af94d3ac52700e0d6f.png)

接着安装telnet 及 telnet-server

![](https://img-blog.csdnimg.cn/direct/9f6632b37da549adbf767bb8b3e251ad.png)

安装完成后设置开机启动及启动telnet服务

![](https://img-blog.csdnimg.cn/direct/46fb146434524846b362d60b27080317.png)

查看telnet 的23端口

![](https://img-blog.csdnimg.cn/direct/0c2c00ed21d94bb3acaafdc3da0f2df0.png)

1.  升级openssl 到1.1.1k版本

升级openssl时需要gcc及依赖，首先检查是否安装了gcc及依赖

![](https://img-blog.csdnimg.cn/direct/22bb7120a94043509ed5a8b868fb915a.png)

本机未安装GCC   需要手动安装  使用opensshGCC 中的rpm文件。

![](https://img-blog.csdnimg.cn/direct/e4d673cf9b89487e82946204737d5da3.png)

![](https://img-blog.csdnimg.cn/direct/b5aca60b4b8d44d5a1fbaff5caa96d9b.png)

显示已安装gcc的版本

1.  安装openssl  使用opt/openssl-1.1.1k.tar.gz   文件

解压

![](https://img-blog.csdnimg.cn/direct/b6a85074a0584bb5a45c7f0ba5a376e6.png)

编译指定安装位置

![](https://img-blog.csdnimg.cn/direct/715bcda65cb8499484319e3ff3d011fc.png)

Make  &&   make  install

切换openssl版本

![](https://img-blog.csdnimg.cn/direct/183a809cbb8345758c8e08c2fb24985f.png)

替换/lib(lib64)和/usr/lib(lib64)和/usr/local/lib(lib64)存在的相应动态库：

![](https://img-blog.csdnimg.cn/direct/56ccb60353ee4365a028517baae57f56.png)

查看openssl版本

![](https://img-blog.csdnimg.cn/direct/3d6fcc790e784d3495b36838e7a58b3b.png)

1.  安装openssh 使用openssh-9.6p1.tar.gz 安装包

备份ssh

![](https://img-blog.csdnimg.cn/direct/308187472af84df5a2fafd568d16f010.png)

开始安装

![](https://img-blog.csdnimg.cn/direct/2c68afb1d3f24d1c82532662e962c5a4.png)

编译时提示如下错误

![](https://img-blog.csdnimg.cn/direct/6ffefb9995644cb6be646c740edfd770.png)

在./configure 命令前增加

![](https://img-blog.csdnimg.cn/direct/2d2c9601892b427ab71b3204b70c8561.png)

编译完成后  make   make  install  

安装完成后修改配置文件

![](https://img-blog.csdnimg.cn/direct/ce3e9661aab941f09414d377c37af4f2.png)

处理升级后ssh -V的openssl版本不正确问题

![](https://img-blog.csdnimg.cn/direct/3a68a5a9c9c6438396a8bc7ec961c87c.png)

重启服务 配置开机启动

![](https://img-blog.csdnimg.cn/direct/0c0a0f193a3e4dd99dcf56298f31c602.png)

重启服务

![](https://img-blog.csdnimg.cn/direct/5628dbea8c3f420db4dc1c8f14edcce7.png)

查验版本

![](https://img-blog.csdnimg.cn/direct/bc970134f8a14f2f93d747eb99d87be7.png)

升级成功

防火墙查看端口是否开放

firewall-cmd --query-port=22/tcp

开启root登录、ssh端口和密码登录：

vim /etc/ssh/sshd\_config

#Port 22 改为 Port 你的ssh端口

#PermitRootLogin prohibit-password 改为 PermitRootLogin yes #运行root账号远程登录

#PasswordAuthentication yes 改为 PasswordAuthentication yes #开启密码认证

#UsePAM no 改为 UsePAM yes #开启UsePAM登录

systemctl enable sshd

查看ssh服务状态

systemctl status sshd

本文转自 <https://blog.csdn.net/calivnBao/article/details/135931410>，如有侵权，请联系删除。