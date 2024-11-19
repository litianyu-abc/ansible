 

nux系统默认ulimit为1024个访问 用户最多可开启的程序数目。一般一个端口（即一个进程）的最高连接为2的16次方65536

通过这个命令 ulimit -n 可以看到默认值为1024

查看全局文件句柄数限制(系统支持的最大值)  
cat /proc/sys/fs/file-max  
查看每个进程文件句柄数限制

ulimit -n

+ 第一步

修改/etc/security/limits.conf文件，在文件中添加如下行(\*指代系统用户名)，修改Linux系统对用户的关于打开文件数的软限制和硬限制

```sh
*               soft    nofile          65535
*               hard    nofile          65535
*               hard    nproc           65535
*               soft    nproc           65535
root            soft    nofile          65535
root            hard    nofile          65535
root            hard    nproc           65535
root            soft    nproc           65535
```

+ 第二步

修改/etc/pam.d/su 、/etc/pam.d/common-session、/etc/pam.d/login，在文件中添加如下行：

> sudo vim /etc/pam.d/su
> 将 pam_limits.so 这一行注释去掉 ，保存修改
> 重起系统
> sudo vim /etc/pam.d/common-session
> 加上以下一行并保存
> session required pam_limits.so
> 打开/etc/pam.d/su，发现是包含/etc/pam.d/common-session这个文件的，所以修改哪个文件都应该是可以的

这个觉得修改su这个文件比较好，取消注释就OK了，不容易出错。

> session required /lib/security/pam\_limits.so
> #如果是64bit系统的话，应该为 :
> session required /lib64/security/pam\_limits.so

 + 第三步
 + 修改/etc/sysctl.conf文件，在文件中(清除文件原始内容（或者在原有的基础上添加，我是这么干的）)添加如下行（修改网络内核对TCP连接的有关限制）。

```sh
net.ipv4.ip_local_port_range = 1024 65535
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_sack = 0
net.core.netdev_max_backlog = 30000
net.ipv4.tcp_no_metrics_save=1
net.core.somaxconn = 262144
net.ipv4.tcp_syncookies = 0
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
```

    + 第四步
    + 执行如下命令（使上述设置生效）：

> /sbin/sysctl -p /etc/sysctl.conf
> /sbin/sysctl -w net.ipv4.route.flush=1

 + 第五步
 + 执行如下命令（linux系统优化完网络必须调高系统允许打开的文件数才能支持大的并发，默认1024是远远不够的）：

> echo "ulimit -HSn 65535" >> /etc/rc.local
> echo "ulimit -HSn 65535" >>/root/.bash\_profile
> echo "ulimit -SHn 65535" >> /etc/profile
> ulimit -SHn 65535

 + 第六步
 + 重启机器

> 通过修改，tcp可以达到65535个连接完全没有问题
>
> 通过这个命令 ulimit -n 可以看到值改为65535了，也就是说现在最多支持65536个tcp socket连接了
>
> 查看当前有多少个TCP连接到当前服务器命令:netstat -antp |grep -i est |wc -l