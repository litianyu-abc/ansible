 

随着[centos](https://so.csdn.net/so/search?q=centos&spm=1001.2101.3001.7020)停止更新与维护，官方的yum仓库也清空，国内第三方同步官方的仓库也是空的了，只能去网上找包来更新。

[https://linux.cc.iitk.ac.in/mirror/centos/elrepo/kernel/el7/x86\_64/RPMS/](https://linux.cc.iitk.ac.in/mirror/centos/elrepo/kernel/el7/x86_64/RPMS/ "https://linux.cc.iitk.ac.in/mirror/centos/elrepo/kernel/el7/x86_64/RPMS/")

这个仓库有我们需要的包

kernel-lt-5.4.99-1.el7.elrepo.x86\_64.rpm

kernel-lt-devel-5.4.99-1.el7.elrepo.x86\_64.rpm

1.包安装

\[root@rke2-n004 ~\]# cd rpm/  
\[root@rke2-n004 rpm\]# rpm -ivh \*  
警告：kernel-lt-5.4.99-1.el7.elrepo.x86\_64.rpm: 头V4 [DSA](https://so.csdn.net/so/search?q=DSA&spm=1001.2101.3001.7020)/SHA1 Signature, 密钥 ID baadae52: NOKEY  
准备中...                          ################################# \[100%\]  
正在升级/安装...  
   1:kernel-lt-devel-5.4.99-1.el7.elre################################# \[ 50%\]  
   2:kernel-lt-5.4.99-1.el7.elrepo    ################################# \[100%\]  
\[root@rke2-n004 rpm\]#   
 

2\. 检查内核列表

\[root@rke2-n004 rpm\]#   
\[root@rke2-n004 rpm\]#  awk -F\\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg  
0 : CentOS [Linux](https://so.csdn.net/so/search?q=Linux&spm=1001.2101.3001.7020) (5.4.99-1.el7.elrepo.x86\_64) 7 (Core)  
1 : CentOS Linux (3.10.0-1160.el7.x86\_64) 7 (Core)  
2 : CentOS Linux (0-rescue-9a335a73f0064595b5f8f3d10e023fd0) 7 (Core)  
 

3.  修改/etc/default/grub 中GRUB\_DEFAULT=0

4.重新生成grub文件

\[root@rke2-n004 rpm\]# grub2-mkconfig -o /boot/grub2/grub.cfg  
Generating grub configuration file ...  
Found linux image: /boot/vmlinuz-5.4.99-1.el7.elrepo.x86\_64  
Found initrd image: /boot/initramfs-5.4.99-1.el7.elrepo.x86\_64.img  
Found linux image: /boot/vmlinuz-3.10.0-1160.el7.x86\_64  
Found initrd image: /boot/initramfs-3.10.0-1160.el7.x86\_64.img  
Found linux image: /boot/vmlinuz-0-rescue-9a335a73f0064595b5f8f3d10e023fd0  
Found initrd image: /boot/initramfs-0-rescue-9a335a73f0064595b5f8f3d10e023fd0.img

5.重新启动  
Last login: Fri Aug  2 11:15:49 2024 from 192.168.1.2  
\[root@rke2-n004 ~\]#   
\[root@rke2-n004 ~\]# uname -r  
5.4.99-1.el7.elrepo.x86\_64  
 

内核升级成功了，但是安全问题需要自己承担。

本文转自 <https://blog.csdn.net/qn0007/article/details/140018597>，如有侵权，请联系删除。