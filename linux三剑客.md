# linux三剑客

![img](https://img-blog.csdnimg.cn/direct/4bf80b0d4f644134a8fb6068775e9980.png)

```shell
在Linux下一切皆为文件，所以在Linux下的操作就是对文件的操作，为了更方便对文件操作，需要用到“三剑客”命令。
```

1、文本处理能力：

```shell
 grep：常用于在文件中搜索指定的字符串，查找。
 sed：常用于对文本进行替换和编辑，修改。
 awk：常用于对文本进行分析和处理，分段，处理文本数据。
```

2，系统管理能力：Linux三剑客在系统管理中有着广泛的应用，可以用于日志分析，数据提取，批量处理等任务，提高系统管理效率

 3，脚本编程功能：grep，sed，awk可以通过脚本编程的方式进行批量处理和自动化操作，可以白那些复杂的文本处理程序和数据分析程序

4，提高工作效率：掌握Linux三剑客可以提高工作效率，特别是对于需要处理大量文本数据的工作，可以节省大量的时间和精力

## **grep：**文本搜索工具，用于过滤、搜索特定的字符。

**解释：**可以帮助用户快速定位包含特定内容的文件或者行。

```shell     
常用的grep参数：                           
        -i：忽略大小写                         
        -r：递归搜索子目录                  
        -n：显示匹配行的行号                 
        -v：显示不包含匹配文本的行   
        -c：显示匹配的行数   
        -V：表示排除匹配结果；
        -n：表示显示匹配行与行号；
        -i：表示不区分大小写，即忽略大小写进行匹配；
        -E：表示支持使用扩展的正则表达式元字符。
        -w：表示只匹配过滤到的单词；
        -o：表示只输出匹配到的内容；
        --color=auto：表示为过滤结果添加颜色。
```

**grep实战场景**---

以下通过实战让大家对grep招式不再陌生。

1）**找出文件haodao.txt中与haodao有关的内容，并且忽略大小写**，命令如下：

```shell
[root@haodaolinux1 ~]# grep -i "haodao" haodao.txt  HAODAOhaodao1:x:1000:1000::/home/haodao1:/bin/bashhaodao2:x:1002:1002::/home/haodao2:/bin/bashhaodao3:x:1003:1003::/home/haodao3:/bin/bash
```

2）**找出文件haodao.txt中与haodao有关的内容，并且忽略大小写，加上行号显示**，命令如下：

```shell
[root@haodaolinux1 ~]# grep -i -n "haodao" haodao.txt 6:HAODAO29:haodao1:x:1000:1000::/home/haodao1:/bin/bash35:haodao2:x:1002:1002::/home/haodao2:/bin/bash36:haodao3:x:1003:1003::/home/haodao3:/bin/bash
```

3）**统计出文件haodao.txt中与haodao有关内容的行，忽略大小写，并且只显示行数**，命令如下：

```shell
[root@haodaolinux1 ~]# grep -i -c "haodao" haodao.txt
```

4）**找出文件haodao.txt中存在的空行，并且将行号打印出来**，命令如下：

```shell
[root@haodaolinux1 ~]# grep "^$" haodao.txt -n
```

5）**找出文件haodao.txt中存在的空行，并且将空行排除，打印除空行外的内容**，命令如下：

```shell
[root@haodaolinux1 ~]# grep "^$" haodao.txt 
-n 
-v1:root:x:0:0:root:/root:/bin/bash2:ROOT:x:1:1:bin:/bin:/sbin/nologin3:root5:ROOT6:HAODAO7:haohao9:haohaolinux10:daemon:x:2:2:daemon:/sbin:/sbin/nologin11:#linux13:#1234567815:adm:x:3:4:adm:/var/adm:/sbin/nologin16:lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin17:sync:x:5:0:sync:/sbin:/bin/sync18:shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown19:halt:x:7:0:halt:/sbin:/sbin/halt20:mail:x:8:12:mail:/var/spool/mail:/sbin/nologin21:operator:x:11:0:operator:/root:/sbin/nologin22:games:x:12:100:games:/usr/games:/sbin/nologin23:ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin24:nobody:x:99:99:Nobody:/:/sbin/nologin25:avahi-autoipd:x:170:170:Avahi IPv4LL Stack:/var/lib/avahi-autoipd:/sbin/nologin26:systemd-bus-proxy:x:999:997:systemd Bus Proxy:/:/sbin/nologin27:systemd-network:x:998:996:systemd Network Management:/:/sbin/nologin28:dbus:x:81:81:System message bus:/:/sbin/nologin29:polkitd:x:997:995:User for polkitd:/:/sbin/nologin30:tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin31:postfix:x:89:89::/var/spool/postfix:/sbin/nologin32:sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin33:haodao1:x:1000:1000::/home/haodao1:/bin/bash34:mysql:x:1001:1001::/home/mysql:/sbin/nologin35:zabbix:x:996:992:Zabbix Monitoring System:/var/lib/zabbix:/sbin/nologin36:apache:x:48:48:Apache:/opt/rh/httpd24/root/usr/share/httpd:/sbin/nologin37:ntp:x:38:38::/etc/ntp:/sbin/nologin38:tcpdump:x:72:72::/:/sbin/nologin39:haodao2:x:1002:1002::/home/haodao2:/bin/bash40:haodao3:x:1003:1003::/home/haodao3:/bin/bash
```

6）**找出文件haodao.txt中，没有以#开头的行，并且也不是空行的内容**，命令如下：

```shell
grep "^#" haodao.txt -v |grep "^$" -v
```

**grep正则表达式元字符集整理：**

```shell
^ ：锚定行的开始 如：’^grep’匹配所有以grep开头的行。

$ ：锚定行的结束 如：’grep$’匹配所有以grep结尾的行。

. ：匹配一个非换行符的字符 如：’gr.p’匹配gr后接一个任意字符，然后是p。

* ：匹配零个或多个先前字符 如：’*grep’匹配所有一个或多个空格后紧跟grep的行。

[] ：匹配一个指定范围内的字符，如'[Gg]rep’匹配Grep和grep。

[^] ：匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep’匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。

.* ：一起用代表任意字符。

.. ：标记匹配字符，如’love’，love被标记为1。

\<word ：以某单词开头

word\> ：以某单词结尾

x/{m/} ：重复字符x，m次，如：’0\{5\}’匹配包含5个o的行。

x\{m,\} ：重复字符x,至少m次，如：’o\{5,\}’匹配至少有5个o的行。

x\{m,n\} ：重复字符x，至少m次，不多于n次，如：’o\{5,10\}’匹配5–10个o的行。

\w ：匹配文字和数字字符

\b ：单词锁定符，如: ‘\bgrep\b’只匹配grep。
```

**grep结合正则表达式用法：**

1)**在文件haodao.txt中找出以haodao结尾的行**，命令如下：

```shell
[root@haodaolinux1 ~]# grep  -n "haodao$" haodao.txt   
21:operator:x:11:0:operator:/root:/sbin/haodao
28:dbus:x:81:81:System message bus:/:/sbin/haodao
32:sshd:x:74:74:Privilege-separated SSH:haodao
```

2）**找出文件haodao.txt中包含大写字母有关的行**，命令如下：

```shell
[root@haodaolinux1 ~]# grep  -n [A-Z] haodao.txt
2:ROOT:x:1:1:bin:/bin:/sbin/nologin5:ROOT6:HAODAO23:ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin24:nobody:x:99:99:Nobody:/:/sbin/nologin25:avahi-autoipd:x:170:170:Avahi IPv4LL Stack:/var/lib/avahi-autoipd:/sbin/nologin26:systemd-bus-proxy:x:999:997:systemd Bus Proxy:/:/sbin/nologin27:systemd-network:x:998:996:systemd Network Management:/:/sbin/nologin28:dbus:x:81:81:System message bus:/:/sbin/haodao29:polkitd:x:997:995:User for polkitd:/:/sbin/nologin30:tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin32:sshd:x:74:74:Privilege-separated SSH:haodao
```



**举例1：**使用参数 ***\*- i ：\****参数忽略大小写。***\*比如筛选出关于root的内容，不区分大小写\****

```shell
[root@localhost ~]# grep -i "ROOT" /etc/passwd  //他会忽略查找内容的大小写
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```

**举例2：**使用参数 ***\*- n ：\****显示匹配行的行号。**比如查找关于root的内容并显示内容行号**

```shell
[root@localhost ~]# grep -n "root" /etc/passwd
1:root:x:0:0:root:/root:/bin/bash
10:operator:x:11:0:operator:/root:/sbin/nologin
```

**举例3：**配合其他一系列命令使用，**比如配合管道符筛选出/etc/passwd中带root关键词的内容**

- **当然这个时候grep也能配合参数使用，比如显示行号**

```shell
[root@localhost ~]# cat  /etc/passwd  | grep -n "root"
1:root:x:0:0:root:/root:/bin/bash
10:operator:x:11:0:operator:/root:/sbin/nologin
```

**举例4：**使用参数 **- v：**显示不包含匹配文本的行 

> 回顾：**"^word"：**表示以word开头
>
> ​      **"word$"：**表示与word结尾
>
> ​       **"^$"：**表示空行

在阅读Linux文件的时候发现了某些问题，**比如以#开头，空行的内容影响阅读，可以筛选**

```shell
[root@localhost ~]# grep -v "^#" /etc/yum.conf | grep -nv "^$"    //参数-v配合-n使用
1:[main]
2:cachedir=/var/cache/yum/$basearch/$releasever
3:keepcache=0
4:debuglevel=2
5:logfile=/var/log/yum.log
6:exactarch=1
7:obsoletes=1
8:gpgcheck=1
9:plugins=1
10:installonly_limit=5
11:bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
12:distroverpkg=centos-release
```

**举例5：**匹配以root和hzx开头的行，并显示对应的行号

```shell
[root@localhost ~]# cat /etc/passwd | grep -n "^\(root\|hzx\)"
1:root:x:0:0:root:/root:/bin/bash
44:hzx:x:1000:1000:hzx:/home/hzx:/bin/bash
 
解释：在正则表达式中,\( 和 \) 用来表示分组，\| 用来表示逻辑或。
```

**举例6：\**多文件匹配。\****多文件匹配时，第一位会告诉你属于什么文件，也能配合参数使用

```shell
[root@localhost ~]# grep -n  "root" /etc/passwd /etc/group
/etc/passwd:1:root:x:0:0:root:/root:/bin/bash
/etc/passwd:10:operator:x:11:0:operator:/root:/sbin/nologin
/etc/group:1:root:x:0:
```



## sed：常用于 编辑和替换

​        sed是一种用来处理文本的工具，它一次处理一行内容。它会把当前处理的行存储在一个临时的缓冲区中，然后对这个缓冲区中的内容进行处理，处理完成后把结果输出到屏幕上。接着再处理下一行，直到文件结束。使用sed不会改变原始文件的内容，除非你使用重定向来保存输出。

在Linux中，sed命令最常用的参数包括：                                                           

```shell
     -e：允许对输入的文本进行多个编辑操作。                                       
     -s：替换指定字符                                                                               
     -d：删除操作                                                                                      
     -i：直接修改文件内容，而不是仅仅输出到屏幕。                             
     -n：禁止默认的输出，只输出经过sed处理的内容。                         
      g：表示行内全面替换
      p：表示打印行       
```

1、**sed常用语法如下：**

```
sed [选项参数] [sed内置命令字符] 文件
```

1）**sed常用参数选项**如下：

```shell
-n：表示取消默认的sed输出，通常与sed内置命令p一起使用；
-i：表示直接将修改结果写入文件，如果不加-i，sed修改的是内存数据；
-e：表示多次编辑，不需要管道符号；
-r：表示支持正则表达式；
```

2）**sed内置的命令字符**，主要是用于对文件进行增删改查等操作，常见内置命令字符有：

```shell
a：表示对文本进行追加操作，在指定行后面添加一行或多行文本；
d：表示删除匹配行；
i：表示插入文本，在指定行前添加一行或多行文本；
p：表示打印匹配行内容，通常与-n一同使用；
s/正则/替换内容/g：表示匹配正则内容，然后替换内容（支持正则表达式），结尾g表示全局匹配；
```

3）**sed匹配范围主要有：**

```shell
空地址：表示全文处理；单地址：表示指定文件某一行；
/pattern/:表示被模式匹配到的每一行；范围区间：如12,22表示十二行到二十二行，如12,+5表示第12行向下5行；
步长：如1~2，表示1,3,5,7,9行；2~2，表示2,4,6，8,10行；
```

**2、以下通过实际例子来练习，加深大家对sed命令的用法。**

**准备一个haotest01.txt文件**，文件内容如下：

![img](https://pics7.baidu.com/feed/11385343fbf2b2116d41898785e9bb350dd78e07.jpeg@f_auto?token=7aeb2e69de45d18ef4251e0934a1678f)

**1）通过sed命令输出1行向下4行的数据**，命令如下所示：

```shell
[root@haodaolinux1 ~]# sed "1,+4p" -n haotest01.txt   
He is a good BOY.
she is a girl !
I have a dream.
We will study linux.
we will study Python.
```

**2）通过sed命令输出文件中的4到6行数据**，命令如下所示：

```shell
[root@haodaolinux1 ~]# sed "4,6p" -n haotest01.txt 
We will study linux.
we will study Python.
So we will go study together.
```

**3）通过sed命令输出有study字符串的行**，命令如下所示：

```shell
[root@haodaolinux1 ~]# sed "/study/p" -n haotest01.txt 
We will study linux.
we will study Python.
So we will go study together.
```

**4）通过sed命令将文件中的good修改替换为bad**，并且输出，命令如下所示：

```shell
[root@haodaolinux1 ~]# sed "s/good/bad/g" haotest01.txt 
He is a bad BOY.she is a girl !
I have a dream.
We will study linux.
we will study Python.
So we will go study together.
[root@haodaolinux1 ~]# cat haotest01.txt 
He is a good BOY.she is a girl !
I have a dream.
We will study linux.
we will study Python.
So we will go study together.
```

**此时发现只是修改输出，原文件并没有实际替换成功，这是因为sed只是修改替换内存中的数据，要加上-i才能修改原文件内容**，命令如下所示：

```shell
[root@haodaolinux1 ~]# sed "s/good/bad/g" -i haotest01.txt 
[root@haodaolinux1 ~]# cat haotest01.txt                   
He is a bad BOY.she is a girl !
I have a dream.
We will study linux.
we will study Python.
So we will go study together.
[root@haodaolinux1 ~]#
```

**如果需要在一行中多次编辑替换多个地方的数据，则要在每个sed编辑语句前加带-e选项参数。**

**5）通过sed命令在文件的第3行下追加一行数据，内容为：we he he**，命令如下所示：

```shell
root@haodaolinux1 ~]# sed "3a we he he" -i haotest01.txt
```

替换后原文件内容如下图所示：

![img](https://pics0.baidu.com/feed/d0c8a786c9177f3e2d5c8da320a6e5ca9e3d56ae.jpeg@f_auto?token=d3f3e64bba5ad466b6837b59fae4735f)

这里是通过内置命令符**a**在指定行下面追加，如果需要在指定行上面加，则要内置命令符**i**

如果需要在指定行下追加多行数据，可以在数据中需要换行的地方添加换行符号**\n**

**6)sed匹配范围空地址的使用，即通过sed命令，在每一行的下面追加5个\*，**命令如下所示：

```shell
[root@haodaolinux1 ~]# sed "a *****" -i haotest01.txt
```

追加后内容如下图所示：

![img](https://pics6.baidu.com/feed/9213b07eca80653830626fcfc7b47f49af3482dd.jpeg@f_auto?token=2a9c9911465a5a6ef5be8f4548718be3)

### **替换操作：s命令**

**举例1：**替换文本中的字符串。

```shell
[root@localhost ~]# sed 's/old/new/' file         
 
解释：匹配file文件中每一行的第一个“old”替换为“new”
```

**举例2：**加上 ***\*- i\**** 选项，将直接修改文件内容，而不是仅仅输出到屏幕。

```shell
[root@localhost ~]# sed -i 's/old/new/' file         
 
解释：-i直接编辑文件，匹配file文件中每一行的第一个“old”替换为“new”
```

**举例3：**使用 ***\*- e\**** 参数：允许对输入的文本进行多个编辑操作。

```shell
[root@localhost ~]# sed -e 's/old/new/' -e 's/book/books/' file
 
解释：-e允许对输入的文本进行多个编辑操作，操作的文档为file
#匹配每一行的第一个“old”替换为“new”，再匹配每一行的第一个“book”替换为“books”
```

### **全面替换：标记 /g**

```shell
[root@localhost ~]# sed -i 's/old/new/g' file         
 
解释：-i直接编辑文件，s为替换，/g会替换每一行中所有的匹配
#将file文件中每一行所有“old”替换为“new”
```

**举例4：**使用 ***\*- n\**** 参数，禁止默认的输出，只输出经过sed处理的内容。

### 打印行：p标记

```shell
[root@localhost ~]# sed -n '1p' /etc/passwd
root:x:0:0:root:/root:/bin/bash
 
解释：打印出文件/etc/passwd第1行内容
```

- ***\*当然，也能够连续打印多行\****

```shell
[root@localhost ~]# sed -n '1,5p' /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
 
 
解释：打印出文件/etc/passwd中1-5行的内容
```

### 删除操作：d命令

**举例5：**删除空白行

```shell
[root@localhost ~]# sed '/^$/d' /etc/named.conf
 
解释：我们拿DNS的主配置文件为例，删除/etc/named.conf文件中的空白行
```

**举例6：**删除文件的第2行文件

```shell
[root@localhost ~]# sed '2d' /etc/named.conf

```

**举例7：**删除文件的第2行到末尾的所有行

```shell
[root@localhost ~]# sed '2,$d' /etc/named.conf

```

**举例8：**删除文件最后一行

```shell
[root@localhost ~]# sed '$d' /etc/named.conf

```

**举例9：**删除文件中所有开头是test的行

```shell
[root@localhost ~]# sed '/^test/d' /etc/named.conf

```

## awk：报告生成器，格式化文本

当处理文本文件时，awk是一个非常强大的工具。它可以用于搜索文件、提取信息、进行格式化输出以及执行计算等操作。

**1、awk语法如下：**

```shell
awk [选项参数] '模式[动作]' 文件
```

这里的动作主要是指awk对文本的一些格式化处理，并且输出格式化处理后的结果，常见有print等，动作写在命令的**{}**里面。

1）**awk常见参数**有：

```shell
-F：表示指定分隔符

-v：表示定义或修改一个awk内置变量；

-f：表示从脚本中读取awk命令；

2）awk有很多内置变量，可以通过man awk查看，以下是常见的内置变量：

$0：表示完整的输入记录；

$n：表示指定分隔符后，当前指定的第n个字段；

FS：表示字段输入分隔符，默认以空格为分隔符；

OFS：表示字段输出分隔符；

NF：表示字段的个数，即以分隔符分割后，当前行一共有多少个字段；

NR：表示当前记录数，即行数；

awk涉及内容或语法比较多，想要一篇文章学完是不带现实，以下通过实例讲解它常见的那些招式，让你在日常工作中足以应付自如。
```

**2、准备一个haotest02.txt文件**，内容如下所示：

![img](https://pics7.baidu.com/feed/4afbfbedab64034fdd47d533ffaaa73c0b551df7.jpeg@f_auto?token=f477311473cbaeb2c9039fe33412fc25)

先执行以下awk命令，如下所示：

```shell
[root@haodaolinux1 ~]# awk '{print $0}' haotest02.txt 
haodao1 haodao2 haodao3
haodao4 haodao5 haodao6
haodao7 haodao8 haodao9
[root@haodaolinux1 ~]# awk '{print $1}' haotest02.txt  
haodao1
haodao4
haodao7
[root@haodaolinux1 ~]# awk '{print $2}' haotest02.txt  
haodao2
haodao5
haodao8
[root@haodaolinux1 ~]#
```

通过以上awk命令，**针对动作{print $0}这里代表一整行输出，针对动作{print $1}代表第一列输出，针对动作{print $2}代表第二列输出**。

**awk默认是以空格符为分隔符**，不管有多少个空格，也会当成一个来处理。

awk是按行处理文件的，一行处理完毕后，再处理下一行，并且是根据用户指定的分隔符去处理，没有指定分隔符的，则默认以空格来当作分隔符。

1）**针对haotest02.txt文件，输出第1列，第3列的内容**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk '{print $1$3}' haotest02.txt  
haodao1 haodao3
haodao4 haodao6
haodao7 haodao9
[root@haodaolinux1 ~]# awk '{print $1 $3}' haotest02.txt 
haodao1 haodao3
haodao4 haodao6
haodao7 haodao9
[root@haodaolinux1 ~]# awk '{print $1,$3}' haotest02.txt  
haodao1 haodao3
haodao4 haodao6
haodao7 haodao9
[root@haodaolinux1 ~]#
```

**对比看出，动作中$1和$3用逗号,可以以空格输出。**

可以添加自定义内容进行输出，如下命令所示：

```shell
[root@haodaolinux1 ~]# awk '{print "第1列为："$1,"第2列为："$3}' haotest02.txt     
第1列为：haodao1 
第2列为：haodao3
第1列为：haodao4 
第2列为：haodao6
第1列为：haodao7 
第2列为：haodao9
[root@haodaolinux1 ~]#
```

2）**针对haotest02.txt文件，输出第1行内容**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk 'NR==1{print $0}' haotest02.txt       
haodao1 haodao2 haodao3
[root@haodaolinux1 ~]#
```

3）**针对haotest02.txt文件，输出第2行到第3行的内容**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk 'NR==2,NR==3{print $0}' haotest02.txt  
haodao4 haodao5 haodao6haodao7 haodao8 haodao9
```

4）**针对haotest02.txt文件，输出第2行到第3行的内容，并且加上行号**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk 'NR==2,NR==3{print NR,$0}' haotest02.txt 
2 haodao4       haodao5 haodao6
3 haodao7       haodao8 haodao9
[root@haodaolinux1 ~]#
```

5）**针对haotest02.txt文件，输出第1列的内容，并且显示出字段数（总列数）**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk '{print $1,NF}' haotest02.txt      
haodao1 3haodao4 3haodao7 3
[root@haodaolinux1 ~]#
```

6）**针对haotest02.txt文件，输出第1列和最后一列的内容**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk '{print $1,$(NF)}' haotest02.txt   
haodao1 haodao3
haodao4 haodao6
haodao7 haodao9
[root@haodaolinux1 ~]#
```

**3、自定义分隔符，进行文件格式化后输出。**

准备一个haotest03.txt文件，内容如下所示：

![img](https://pics7.baidu.com/feed/1f178a82b9014a90b102fc18f91ee71fb21beeca.jpeg@f_auto?token=89d002cf06ab0343e4ab2bf54e734661)

1）**通过指定输入分隔符，如以:为输入分隔符，输出文件第一列的内容**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk -F ":" '{print $1}' haotest03.txt 
root
bin
daemon
mail
ntp
tcpdump
haodao2
haodao3
[root@haodaolinux1 ~]#
```

或者**通过修改系统默认输入分隔符变量（FS），进行输出**，命令如下所示:

```shell
[root@haodaolinux1 ~]# awk -v FS=':' '{print $1}' haotest03.txt     
root
bin
daemon
mail
ntp
tcpdump
haodao2
haodao3
[root@haodaolinux1 ~]#
```

2）**通过修改默认输出分隔符变量（OFS），输出文件第一列和最后一列内容，并且两列之间就是通过输出分隔符隔开的**，命令如下所示：

```shell
[root@haodaolinux1 ~]# awk -F ":" -v OFS='*****' '{print $1,$NF}' haotest03.txt root*****/bin/bashbin*****/sbin/nologindaemon*****/sbin/nologinmail*****/sbin/nologinntp*****/sbin/nologintcpdump*****/sbin/nologinhaodao2*****/bin/bashhaodao3*****/bin/bash
[root@haodaolinux1 ~]#
```

以下是一些基本的使用方法和示例：

**举例1**： 提取特定字段
   如果你有一个包含多个字段的文本文件，你可以使用awk来提取其中的某些字段。

        假设有一个包含学生信息的文件，每行包括学生的姓名、年龄和成绩，你可以使用awk来提取其中的某些字段：

```shell
[root@localhost ~]# awk '{print $1, $3}' students.txt
 
解释：这个命令将打印出每行中的第一个和第三个字段（即姓名和成绩）。
```

**举例2：** ***\*进行计算\****
  你可以使用awk来对文本文件中的数据进行计算。

**假设你有一个包含数字的文件，你可以使用awk来计算它们的总和**

```shell
[root@localhost ~]# awk '{sum += $1} END {print "Total: ", sum}' numbers.txt
 
解释：这个命令将计算文件中所有数字的总和，并打印出结果。
```

**举例3： \**过滤数据\****
  你可以使用awk来过滤文本文件中的数据，只输出符合条件的行或字段。

**比如，你可以只输出成绩大于80分的学生信息**

```shell
[root@localhost ~]#  awk '$3 > 80 {print $1, $3}' students.txt
 
解释：这个命令将只输出成绩大于80分的学生的姓名和成绩。
```



























































































































































































































































