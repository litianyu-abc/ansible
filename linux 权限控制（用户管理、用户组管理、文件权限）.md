 

#### 文章目录

*   *   [1\. 用户管理](#1__8)
    *   *   [1.1 useradd 添加新用户并在home目录下创建对应用户的目录](#11_useradd_home_9)
        *   [1.2 passwd 设置用户密码](#12_passwd__29)
        *   [1.3 id 查看用户是否存在](#13_id__42)
        *   [1.4 cat /etc/passwd 查看创建的用户](#14_cat__etcpasswd__57)
        *   [1.5 su 切换用户](#15_su__66)
        *   [1.6 userdel 删除用户](#16_userdel__87)
        *   [1.7 who 查看登录用户信息](#17_who__121)
        *   [1.8 sudo 设置普通用户具有root权限](#18_sudo_root_147)
        *   [1.9 usermod 修改用户所属的组](#19_usermod__191)
    *   [2\. 用户组管理](#2__218)
    *   *   [2.1 cat /etc/group 查看创建的组](#21_cat__etcgroup__221)
        *   [2.2 groupadd 新增组](#22_groupadd__229)
        *   [2.3 groupdel 删除组](#23__groupdel__241)
        *   [2.4 groupmod 修改组](#24_groupmod__250)
    *   [3\. 文件权限](#3___263)
    *   *   [3.1 文件属性](#31__264)
        *   [3.2 chmod 改变权限](#32__chmod__307)
        *   [3.3 chown 改变文件或目录所有者](#33_chown__372)
        *   [3.4 chgrp 改变文件或目录所属组](#34_chgrp__400)

* * *

### 1\. 用户管理

#### 1.1 useradd 添加新用户并在home目录下创建对应用户的目录

```bash
useradd 用户名			（功能描述：添加新用户）
useradd -g 组名 用户名	（功能描述：添加新用户到某个组）
id 	用户名               （检测用户）
```

注：管理用户的文件是/etc/passwd  
![在这里插入图片描述](https://img-blog.csdnimg.cn/17100d13cd624bb280c02d475ee1b4fe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2G6KGM55uK5LqL6I6r6Zeu5YmN56iL,size_20,color_FFFFFF,t_70,g_se,x_16)

**如：**  
添加一个rng用户

```bash
useradd rng
ll /home/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d634f162bdb54e77b528d9c6867e4b3d.png)

#### 1.2 passwd 设置用户密码

```bash
passwd 用户名	（功能描述：设置用户密码）
```

**如：**  
设置rng用户的密码

```bash
 passwd rng
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc9bdf26b3d54696adf5ef9ee1ba3f05.png)

#### 1.3 id 查看用户是否存在

```bash
	id 用户名
```

**如：**  
查看rng用户是否存在

```bash
id rng
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/73ff25d104594eab9b98d226556a6f73.png)

* * *

#### 1.4 cat /etc/passwd 查看创建的用户

查看创建的rng用户：

```bash
cat  /etc/passwd |grep rng
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/aba71547b51149beb76ba86fca435d47.png)

#### 1.5 su 切换用户

```bash
su 用户名称   		（功能描述：切换用户，只能获得用户的执行权限，不能获得环境变量）
su - 用户名称		（功能描述：切换到用户并获得该用户的环境变量及执行权限）
exit                （功能描述：登出）
```

**如：**  
从root用户切换到rng用户的不同方式：

```bash
su rng
echo $PATH
exit
su - rng
echo $PATH
exit
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f2b05791e6e94a7e9306fd4939891836.png)

#### 1.6 userdel 删除用户

```bash
userdel  用户名			（功能描述：删除用户但保存用户主目录）
userdel -r 用户名		（功能描述：用户和用户主目录，都删除）
```

**如：**

（1）删除rng用户但保存用户主目录:

```bash
userdel rng
cat /etc/passwd |grep rng
ll /home/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e9c0900c1692450396ad8d15a2b89d7c.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f1303f091f5411888acbccac5479a12.png)

（2）删除rng用户和用户主目录

```bash
useradd rng
id rng 
ll /home/
userdel -r rng
id rng
ll /home/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f9120958693c4f64b6261f61033255de.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2G6KGM55uK5LqL6I6r6Zeu5YmN56iL,size_15,color_FFFFFF,t_70,g_se,x_16)

#### 1.7 who 查看登录用户信息

```bash
whoami			（功能描述：显示自身用户名称）
who am i		（功能描述：显示登录用户的用户名以及登陆时间）
```

**如：**

（1）显示当前登录用户名称

```bash
whoami
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0f2c617278a342058d0eb7bc75cc9e40.png)

（2）显示登录用户的用户名和登录时间

```bash
 who am i
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/12ef5c55498a4f3f9644388a0d6a609a.png)

#### 1.8 sudo 设置普通用户具有[root权限](https://so.csdn.net/so/search?q=root%E6%9D%83%E9%99%90&spm=1001.2101.3001.7020)

（1）添加rng用户，并对其设置密码

```bash
useradd rng
passwd rng
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3fe5f806cc614506bebe0406ed502b6d.png)  
（2）修改配置文件

```bash
vim /etc/sudoers
```

修改 /etc/sudoers 文件，找到下面一行  
![在这里插入图片描述](https://img-blog.csdnimg.cn/82c6cdf61dc346daaf56f1f55f646f2a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2G6KGM55uK5LqL6I6r6Zeu5YmN56iL,size_17,color_FFFFFF,t_70,g_se,x_16)

在root下面添加一行

```bash
rng  ALL=(ALL)     ALL
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d522792aa90d472c8d8d0b12b36bcba6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2G6KGM55uK5LqL6I6r6Zeu5YmN56iL,size_16,color_FFFFFF,t_70,g_se,x_16)

或者配置成采用sudo命令时，不需要输入密码

```bash
rng ALL=(ALL)     NOPASSWD:ALL
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/67f7021f9f7e443dbf90b2602b089187.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2G6KGM55uK5LqL6I6r6Zeu5YmN56iL,size_16,color_FFFFFF,t_70,g_se,x_16)

wq!强制退出，修改完毕，切换到rng用户，在执行的命令前添加 sudo ，即可获得root权限进行操作

**如：**

用普通用户rng创建删除用户mm

```bash
sudo useradd mm
sudo userdel -r mm
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/455681a79055458b899a70a5a3d61ba9.png)

#### 1.9 [usermod](https://so.csdn.net/so/search?q=usermod&spm=1001.2101.3001.7020) 修改用户所属的组

```bash
usermod -g 用户组 用户名
```

| 选项 | 功能 |
| --- | --- |
| \-g | 修改用户的初始登录组，给定的组必须存在。默认组id是1 |

**如：**

将用户加入到用户组

```bash
usermod -g root rng
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/88af4d3e09c74384824a9032f3bdf60b.png)

* * *

### 2\. 用户组管理

  每个用户都有一个用户组，系统可以对一个用户组中的所有用户进行集中管理。一般Linux下的用户属于与它同名的用户组，这个用户组在创建用户时同时创建。  
  用户组的管理涉及用户组的添加、删除和修改。组的增加、删除和修改实际上就是对/etc/group文件的更新。

#### 2.1 cat /etc/group 查看创建的组

```bash
 cat  /etc/group
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a62c4f059bb4f2bbb9ced498791db2d.png)

#### 2.2 [groupadd](https://so.csdn.net/so/search?q=groupadd&spm=1001.2101.3001.7020) 新增组

```bash
groupadd 组名
```

**如：**  
添加组lpl:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b1dffe686c14436ad98706aefbecede.png)

#### 2.3 groupdel 删除组

```bash
groupdel 组名
```

**如：**  
删除组lpl:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/91b4799523994d6887f2f7f7820f8f62.png)

#### 2.4 groupmod 修改组

```bash
groupmod -n 新组名 老组名
```

**如：**  
修改组名lpl为lck:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/532dbb0cb80e48c8b435c1a1900f0b2c.png)

* * *

### 3\. 文件权限

#### 3.1 文件属性

  Linux系统是一种典型的多用户系统，不同的用户处于不同的地位拥有不同的权限。为了保护系统的安全性，Linux系统对不同的用户访问同一文件（包括目录文件）的权限做了不同的规定。可以使用ll命令来显示一个文件的属性以及文件所属的用户和组  
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf36ea42d309408688451d44f0e898d7.png)

```bash
第1列是文件类型加权限
第2列是硬链接的引用次数
第3列是文件的拥有者
第4列是文件的拥有者所在组名
第5列是文件所占有的字节数
第6列是文件最后修改时间
第7列是文件名
```

**1）从左到右的10个字符代表不同的权限，没有权限就会出现减号**

![在这里插入图片描述](https://img-blog.csdnimg.cn/7ae2081f48694f1bbc4fdac02b3336de.png)  
从左至右用0-9数字来表示:

```bash
（1）0首位表示类型
	 - 代表文件
	 d 代表目录
	 l 链接
（2）第1-3位确定属主（该文件的所有者）拥有该文件的权限
（3）第4-6位确定属组（所有者的同组用户）拥有该文件的权限
（4）第7-9位确定其他用户拥有该文件的权限
```

**2）rxw作用文件和目录的不同解释**

```bash
（1）作用到文件
	[r]代表可读(read): 可以读取，查看
	[w]代表可写(write): 可以修改，但是不代表可以删除该文件，删除一个文件的前提条件是对该文件所在的目录有写权限，才能删除该文件.
	[x] 代表可执行(execute):可以被系统执行
	
（2）作用到目录：
	[r]代表可读(read): 可以读取，ls查看目录内容
	[w 代表可写(write): 可以修改，目录内创建+删除+重命名目录
	[x]代表可执行(execute):可以进入该目录
```

#### 3.2 chmod 改变权限

**第一种方式变更权限： u:所有者 g:所有组 o:其他人 a:所有人(u、g、o的总和)**

```
chmod  [{ugoa}{+-=}{rwx}] 	文件或目录
```

**第二种方式变更权限**

```bash
chmod  [mode=421]  [文件或目录]
```

其中数字 4 、2 和 1表示读、写、执行权限即 r=4，w=2，x=1 ，其他的权限组合可使用：

```c
rwx = 4 + 2 + 1 = 7
rw = 4 + 2 = 6
rx = 4 +1 = 5
```

举例:**chmod 600 文件名** (等价于 chmod u=rw,g=—,o=— 文件名)

**如：**  
当前profile属于root用户root组，root用户具有读写权限，root组具有读权限，其他用户具有读权限  
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f0a1643e70c474a8e4e355384a3f91f.png)

```
（1）修改文件使其所属主用户具有执行权限
```

```bash
chmod u+x profile
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8bc6a6357c714063adba0468461d56a9.png)

（2）修改文件使其所属组用户具有执行权限

```bash
chmod g+x profile
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e88a56a5e2614eefb40e5b060ca2cb79.png)

（3）修改文件所属主用户执行权限,并使其他用户具有执行权限

```bash
chmod u+x,o+x profile
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9fa99bb9855a4e1eaa85beb939904e43.png)

（4）采用数字的方式，设置文件所有者、所属组、其他用户都具有可读可写可执行权限

```bash
chmod 777 profile
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1920a14ca405424b9e510fa1f9547855.png)

（5）修改整个文件夹里面的所有文件的所有者、所属组、其他用户都具有可读可写可执行权限。

```bash
chmod -R 777 m/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/535414e216b443a085a4c77e6ef14b6c.png)

#### 3.3 chown 改变文件或目录所有者

```bash
chown [选项] [用户:用户组] [文件或目录]		（功能描述：改变文件或者目录的所有者）
```

| 选项 | 功能 |
| --- | --- |
| \-R | 递归操作 |

**如：**

（1）修改文件所有者

```bash
 chown  rng profile
 ll | grep profile
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dc19df1759f24a40916ac6b95175abd6.png)

（2）递归改变文件所有者和所有组

```bash
chown -R rng:lck m/
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/643765ea9d5b47258702f28ecd8633b6.png)

#### 3.4 chgrp 改变文件或目录所属组

```bash
chgrp [最终用户组] [文件或目录]	（功能描述：改变文件或者目录的所属组）
```

**如：**

修改文件的所属组

```bash
 chgrp root a.txt
 ll | grep a.txt
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0835d1ab82674847827405a18e7104e2.png)

