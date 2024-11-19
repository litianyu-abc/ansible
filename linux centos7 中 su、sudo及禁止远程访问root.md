# linux centos7 中 su、sudo及禁止远程访问root

## 一、 su命令

### 1.切换用户su - fxq

su命令后带"- ",表示环境变量一起切换过去

```shell
[root@VM_46_188_centos ~]# whoami root
[root@VM_46_188_centos ~]#
Last login: Tue Aug  8 22:30:12 CST 2017 on pts/0
[fxq@VM_46_188_centos ~]$ pwd
/home/fxq
[fxq@VM_46_188_centos ~]$ su
Password: 
[root@VM_46_188_centos fxq]# pwd/home/fxq
[root@VM_46_188_centos fxq]# cd 
[root@VM_46_188_centos ~]# 
[fxq@VM_46_188_centos root]$ pwd
/root
[fxq@VM_46_188_centos root]$
```

### 2. root 以user2身份执行命令：

**su - -c "touch /tmp/user2.txt" user2**

```shell
[fxq@VM_46_188_centos root]$ su - -c "touch /tmp/user2.txt" user2
Password: 
[fxq@VM_46_188_centos root]$ exit
[root@VM_46_188_centos ~]# su - -c "touch /tmp/user2.txt" user2
[root@VM_46_188_centos ~]# ls -l /tmptotal 200-rw-r--r-- 1 root   root      238 Aug  8 15:15 1.txt
-rwxr--r-- 1 root   root        0 Aug  7 22:25 2.txt
-rwxr-xr-x 1 root   root      743 Jul  5 17:13 aliyun-yum-install.sh
drwxr-xr-x 4 root   root     4096 Aug  6 21:32 fxq
-rw-r--r-- 1 root   root      486 Aug 13 05:00 ls.log1
-rwxr-xr-x 1 root   root   117616 Aug  6 21:09 ls2
-rw-r--r-- 1 root   root     1742 Aug  6 21:20 passwd
-rw-r--r-- 1 root   root     1742 Aug  6 21:27 passwd2
-rw------- 1 daemon daemon     33 Aug 13 14:36 sess_1b99a57326be2b43a
e545ca31df73d40
-rw------- 1 daemon daemon     33 Aug 13 03:18 sess_2cc7b36f29dd4273f
c82a383b7e05593
-rw------- 1 daemon daemon     33 Aug 13 14:36 sess_2e809bf78820261ef35e4004dbc8aa9f
-rw------- 1 daemon daemon     33 Aug 13 23:27 sess_a5aa9fa1bd0986f8f968e1283c28bf08
-rw-rw-r-- 1 user2  user2   0 Aug 13 23:30 user2.txt
[root@VM_46_188_centos ~]#
```

### 3. 系统中环境变量文件备用文件存储位置

ls -la /etc/skel/ 里面有环境变量文件

如果用户家目录中没有环境变量文件可以从些文件夹中copy。然后改变对应用户的所属主和所属组.

```shell
[root@VM_46_188_centos ~]# ls -la /etc/skel/total 28drwxr-xr-x.  2 root root  4096 Nov 10  2016 .
drwxr-xr-x. 95 root root 12288 Aug 13 23:33 ..
-rw-r--r--   1 root root    18 Aug  3  2016 .bash_logout
-rw-r--r--   1 root root   193 Aug  3  2016 .bash_profile
-rw-r--r--   1 root root   231 Aug  3  2016 .bashrc
[root@VM_46_188_centos ~]#
```

模拟环境变量文件丢失后恢复：

```shell
[root@VM_46_188_centos skel]# rm -rf /home/user111/.bash*
[root@VM_46_188_centos skel]# ls -la /home/user111/       
total 8
drwx------   2 user111 user111 4096 Aug 13 23:42 .
drwxr-xr-x. 17 root    root    4096 Aug 13 23:33 ..
[root@VM_46_188_centos skel]# su - user111
Last login: Sun Aug 13 23:38:18 CST 2017 on pts/0-bash-4.2$ 
-bash-4.2$ 
-bash-4.2$ pwd
/home/user111
-bash-4.2$ logout 

[root@VM_46_188_centos skel]# pwd/etc/skel
[root@VM_46_188_centos skel]# ls -a.  ..  .bash_logout  .bash_profile  .bashrc
[root@VM_46_188_centos skel]# cp .bash* /home/user111/
[root@VM_46_188_centos skel]# ls -a /home/user111/
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc
[root@VM_46_188_centos skel]# su - user111
Last login: Sun Aug 13 23:42:58 CST 2017 on pts/0
[user111@VM_46_188_centos ~]$ 
[user111@VM_46_188_centos ~]$ pwd
/home/user111
[user111@VM_46_188_centos ~]$
```

## 二、 sudo命令

### 1. visudo 打开sudo的配置文件

vi /etc/sudoers.tmp 编辑完没有语法错误提示

```shell
## The COMMANDS section may have other options added to it.#### Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
```

vi 中显示行号: "set nu"

```shell
[fxq@VM_46_188_centos ~]$ ls /root
ls: cannot open directory /root: Permission denied
[fxq@VM_46_188_centos ~]$ sudo ls /root
[sudo] password for fxq: 
#fxq.txt       123.zip		     gdlogo.png,	       2		     gdlogo.png.1,.pub	       2.cap		     gdlogo.png.hard1	       22.txt		     gdlogo.png.soft1.cap	       234		     httpd_process_check.sh1.ipt	       3.txt		     ip.txt1.log.tar      4.txt		     iptables.rules1.log.tar.bz2  [1-3].log	     null1.log.tar.gz   \fxq.txt		     ping_host_alive.sh1.log.tar1.xz  a.out		     sed.txt1.log.xz       anaconda-ks.cfg	     shell1.log.zip      auto_install_lamp.sh  testOS.xsh1.txt	       baidu.png	     weixin111.txt        dir-2017-05-12	     wordpress-4.7.4-zh_CN.zip11111111       ffff		     youjian.sh12	       fxq		     ~iptables_rules123	       fxq-0.xsh
[fxq@VM_46_188_centos ~]$

[fxq@VM_46_188_centos ~]$ ls /root
ls: cannot open directory /root: Permission denied
[fxq@VM_46_188_centos ~]$ sudo ls /root
#fxq.txt       123.zip		     gdlogo.png,	       2		     gdlogo.png.1,.pub	       2.cap		     gdlogo.png.hard1	       22.txt		     gdlogo.png.soft1.cap	       234		     httpd_process_check.sh1.ipt	       3.txt		     ip.txt1.log.tar      4.txt		     iptables.rules1.log.tar.bz2  [1-3].log	     null1.log.tar.gz   \fxq.txt		     ping_host_alive.sh1.log.tar1.xz  a.out		     sed.txt1.log.xz       anaconda-ks.cfg	     shell1.log.zip      auto_install_lamp.sh  testOS.xsh1.txt	       baidu.png	     weixin111.txt        dir-2017-05-12	     wordpress-4.7.4-zh_CN.zip11111111       ffff		     youjian.sh12	       fxq		     ~iptables_rules123	       fxq-0.xsh
[fxq@VM_46_188_centos ~]$
```

### 2. 用户sudo时不用输入密码:

```shell
## The COMMANDS section may have other options added to it.
#### Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

#
# wildcards for entire domains) or IP addresses instead.
# Host_Alias     FILESERVERS = fs1, fs2# Host_Alias     MAILSERVERS = smtp, smtp2
#
# User Aliases
#
# These aren't often necessary, as you can use regular groups
#
# (ie, from files, LDAP, NIS, etc) in this file - just use %groupname## rather than USERALIAS
# User_Alias ADMINS = jsmith, mikem
```

## 三、 限制root远程登录

### 1. 建立用户组

```shell
#
# User Aliases## These aren't often necessary, as you can use regular groups
#
# (ie, from files, LDAP, NIS, etc) in this file - just use %groupname## rather than USERALIAS
# User_Alias ADMINS = jsmith, mikemUser_Alias 
ADMINS = fxq,user111
```

### 2. 切换用户时不需要输入密码

```shell
##      user    MACHINE=COMMANDS#### The COMMANDS section may have other options added to it.#### Allow root to run any commands anywhereroot    ALL=(ALL)       ALLADMINS     ALL=(ALL)    NOPASSWD: /usr/bin/su
```

### 3. 限制root用户远程登录

#### （1）编辑配置文件：

```shell
vi /etc/ssh/sshd_config
#LoginGraceTime 2m
PermitRootLogin no#StrictModes yes#MaxAuthTries 6#MaxSessions 10
```

#### （2）重启sshd服务

```shell
systemctl restart sshd

更改完配置文件后，重启sshd服务，测试root确实无法登录:
```

#### (3) 测试root用户远程登录

```shell
#远程无法登录root，可以通过刚才做的普通用户su - 进入root用户.
[@VM_46_188_centos ~]$ 
[fxq@VM_46_188_centos ~]$ 
[fxq@VM_46_188_centos ~]$ 
[fxq@VM_46_188_centos ~]$ sudo su - 
Last login: Mon Aug 14 00:12:59 CST 2017 on pts/0
Last failed login: Mon Aug 14 00:18:33 CST 2017 from 60.10.158.223 on
 ssh:nottyThere was 1 failed login attempt since the last successful login.
[@VM_46_188_centos ~]# 
[root@VM_46_188_centos ~]#
```

## 四、扩展

1. sudo与su比较 http://www.apelearn.com/bbs/thread-7467-1-1.html

su 和 sudo 命令的区别

su命令就是临时切换用户的工具,su 加参数 -，表示默认切换到root用户，并且改变到root用户的环境；su只适用于一两个人参与管理的系统,多人管理的话不安全。

sudo 授权许可使用的su，也是受限制的su：sudo 执行命令的流程是当前用户切换到root（或其它指定切换到的用户），然后以root（或其它指定的切换到的用户）身份执行命令，执行完成后，直接退回到当前用户；

2. sudo配置文件样例 [www.opensource.apple.com/source/sudo/sudo-16/sudo/sample.sudoers](http://www.opensource.apple.com/source/sudo/sudo-16/sudo/sample.sudoers)

```shell
#
# Sample /etc/sudoers file.
#
# This file MUST be edited with the 'visudo' command as root.
#
# See the sudoers man page for the details on how to write a sudoers file.
#

##
# User alias specification
##
User_Alias	FULLTIMERS = millert, mikef, dowdy
User_Alias	PARTTIMERS = bostley, jwfox, crawl
User_Alias	WEBMASTERS = will, wendy, wim

##
# Runas alias specification
##
Runas_Alias	OP = root, operator
Runas_Alias	DB = oracle, sybase

##
# Host alias specification
##
Host_Alias	SPARC = bigtime, eclipse, moet, anchor:\
		SGI = grolsch, dandelion, black:\
		ALPHA = widget, thalamus, foobar:\
		HPPA = boa, nag, python
Host_Alias	CUNETS = 128.138.0.0/255.255.0.0
Host_Alias	CSNETS = 128.138.243.0, 128.138.204.0/24, 128.138.242.0
Host_Alias	SERVERS = master, mail, www, ns
Host_Alias	CDROM = orion, perseus, hercules

##
# Cmnd alias specification
##
Cmnd_Alias	DUMPS = /usr/sbin/dump, /usr/sbin/rdump, /usr/sbin/restore, \
			/usr/sbin/rrestore, /usr/bin/mt
Cmnd_Alias	KILL = /usr/bin/killCmnd_Alias	PRINTING = /usr/sbin/lpc, /usr/bin/lprm
Cmnd_Alias	SHUTDOWN = /usr/sbin/shutdownCmnd_Alias	HALT = /usr/sbin/halt
Cmnd_Alias	REBOOT = /usr/sbin/reboot
Cmnd_Alias	SHELLS = /sbin/sh, /usr/bin/sh, /usr/bin/csh, /usr/bin/ksh, \
			 /usr/local/bin/tcsh, /usr/bin/rsh, \
			 /usr/local/bin/zsh
Cmnd_Alias	SU = /usr/bin/su
Cmnd_Alias	VIPW = /usr/sbin/vipw, /usr/bin/passwd, /usr/bin/chsh, \
		       /usr/bin/chfn

##
# Override built-in defaults##Defaults               syslog=authDefaults>root          !set_lognameDefaults:FULLTIMERS    !lectureDefaults:millert       !authenticateDefaults@SERVERS       log_year, logfile=/var/log/sudo.log##
# User specification
##

# root and users in group wheel can run anything on any machine as any userroot		ALL = (ALL) ALL
%wheel		ALL = (ALL) ALL

# full time sysadmins can run anything on any machine without a passwordFULLTIMERS	ALL = NOPASSWD: ALL

# part time sysadmins may run anything but need a passwordPARTTIMERS	ALL = ALL

# jack may run anything on machines in CSNETS
jack		CSNETS = ALL

# lisa may run any command on any host in CUNETS (a class B network)
lisa		CUNETS = ALL

# operator may run maintenance commands and anything in /usr/oper/bin/operator	ALL = DUMPS, KILL, SHUTDOWN, HALT, REBOOT, PRINTING,\
		sudoedit /etc/printcap, /usr/oper/bin/

# joe may su only to operatorjoe		ALL = /usr/bin/su operator# pete may change passwords for anyone but root on the hp snakes
pete		HPPA = /usr/bin/passwd [A-z]*, !/usr/bin/passwd root

# bob may run anything on the sparc and sgi machines as any user# listed in the Runas_Alias "OP" (ie: root and operator)
bob		SPARC = (OP) ALL : SGI = (OP) ALL

# jim may run anything on machines in the biglab netgroup
jim		+biglab = ALL

# users in the secretaries netgroup need to help manage the printers
# as well as add and remove users+secretaries	ALL = PRINTING, /usr/bin/adduser, /usr/bin/rmuser

# fred can run commands as oracle or sybase without a passwordfred		ALL = (DB) NOPASSWD: ALL

# on the alphas, john may su to anyone but root and flags are not allowed
john		ALPHA = /usr/bin/su [!-]*, !/usr/bin/su *root*

# jen can run anything on all machines except the ones
# in the "SERVERS" Host_Alias
jen		ALL, !SERVERS = ALL

# jill can run any commands in the directory /usr/bin/, except for# those in the SU and SHELLS aliases.
jill		SERVERS = /usr/bin/, !SU, !SHELLS

# steve can run any command in the directory /usr/local/op_commands/
# as user operator.
steve		CSNETS = (operator) /usr/local/op_commands/

# matt needs to be able to kill things on his workstation when# they get hung.
matt		valkyrie = KILL# users in the WEBMASTERS User_Alias (will, wendy, and wim)
# may run any command as user www (which owns the web pages)
# or simply su to www.
WEBMASTERS	www = (www) ALL, (root) /usr/bin/su www

# anyone can mount/unmount a cd-rom on the machines in the CDROM aliasALL		CDROM = NOPASSWD: /sbin/umount /CDROM,\
		/sbin/mount -o nosuid\,nodev /dev/cd0a /CDROM
```

3. sudo -i 也可以登录到root吗？ http://www.apelearn.com/bbs/thread-6899-1-1.html

```shell
sudo -i: 为了频繁的执行某些只有超级用户才能执行的权限，而不用每次输入密码，可以使用该命令。提示输入密码时该密码为当前账户的密码。没有时间限制。执行该命令后提示符变为“#”而不是“$”。想退回普通账户时可以执行“exit”或“logout” 。
其实，还有几个类似的用法：
sudo /bin/bash   ： 这个命令也会切换到root的bash下，但不能完全拥有root的所有环境变量，比如PATH，可以拥有root用户的权限。这个命令和 sudo -s 是等同的。
sudo -s : 如上
sudo su  ： 这个命令，也是登录到了root，但是并没有切换root的环境变量，比如PATH。
sudo su - :  这个命令，纯粹的切换到root环境下，可以这样理解，先是切换到了root身份，然后又以root身份执行了 su - ，这个时候跟使用root登录没有什么区别。这个结果貌似跟sudo -i 的效果是一样的，但是也有不同，sudo 只是临时拥有了root的权限，而su则是使用root账号登录了linux系统。

所以，我们再来总结一下：
sudo su -  约等于  sudo -i 
sudo -s  完全等于  sudo  /bin/bash  约等于 sudo su sudo 终究被一个"临时权限的帽子"扣住，不能等价于纯粹的登录到系统里。
```

















































































































































































