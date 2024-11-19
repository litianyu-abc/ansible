 

# rsync简介

## 一. rsync介绍

rsync（remote synchronize）是Liunx/Unix下的一个远程数据同步工具。它可通过LAN/WAN快速同步多台主机间的文件和目录，并适当利用rsync算法（差分编码）以减少数据的传输。

rsync算法并不是每一次都整份传输，而是只传输两个文件的不同部分，因此其传输速度相当快。

除此之外，rsync可拷贝、显示目录属性，以及拷贝文件，并可选择性的压缩以及递归拷贝。

### 1\. 工作原理

*   客户端构造FileList，FileList包含了需要与服务器同步的所有文件信息对name->id（id用来唯一表示文件例如MD5）
    
*   客户端将FileList发送到服务器。
    
*   服务器上rsync处理客户端发过来的FileList，构建新的NewFileList。  
    其中根据MD5值比较，删除服务器上已经存在的文件信息对，只保留服务器上不存在或变化的文件。
    
*   客户端得到服务器发送过来的NewFileList，然后把NewFileList中的文件重新传输到服务器。
    

### 2\. 优点

*   可以镜像保存整个目录树和文件系统。
    
*   可以很容易做到保持原来文件的权限、时间、软硬连接等。
    
*   无需特殊权限即可安装。
    
*   快速：第一次同步时rsync复制全部内容，但在下一次值传输修改过的内容
    
*   压缩传输：rysnc在传输的过程中可以实行压缩及解压缩操作，可以使用更少的带宽
    
*   安全：可以使用scp、ssh等方式来进行文件传输
    
*   支持匿名传输，以方便进行网站镜像
    
*   rsync不仅可以远程同步数据（类似于scp），而且可以本地同步数据（类似于cp），做差异同步
    
*   openssh 8.0已经把scp标记为过时不建议使用了。建议用sftp或者rsync替代scp
    

### 3\. 认证方式

rsync有两种常用的认证方式，一种是rsync-daemon方式，另外一种是ssh方式，习惯使用ssh方式。

### （1）rsync-daemon认证

rsync在rsync-daemon认证方式下，默认监听TCP的873端口。

rsync-daemon认证方式是rsync的主要认证方式，这个也是我们经常使用的认证方式。并且也只有在此种模式下，rsync才可以把密码写入到一个文件中。

###### 注意：

*   rsync-daemon认证方式，需要服务器和客户端都安装rsync服务，并且只需要rsync服务器端启动rsync，同时配置rsync配置文件。
*   客户端启动不启动rsync服务，都不影响同步的正常进行。

### （2）ssh认证

rsync在ssh认证方式下，可通过系统用户进行认证，即在rsync上通过ssh隧道进行传输，类似于scp工具，此时同步操作不在局限于rsync中定义的同步文件夹。

注意：

*   ssh认证方式， **不需要服务器和客户端配置rsync配置文件，只需要双方都安装rsync服务，并且也不需要双方启动rsync**。
    
*   若rsync服务端SSH为标准端口，此时rsync使用方式如下：
    

```handlebars
# 将本机（10.7.2.230）上/root目录及目录下的文件同步到服务器10.7.2.231上的/root/backup目录下
# 在10.7.2.230上执行以下命令：
rsync -avuz /root root@10.7.2.231:/root/backup
# 带密码执行
sshpass -p '231机器root的密码' rsync -avuz /root root@10.7.2.231:/root/backup
```

*   若rsync服务端SSH为非标准端口，可通过rsync的-e参数进行端口指定（如ssh端口号为10022），使用方式如下：

```handlebars
# 将本机（10.7.2.230）上/root目录及目录下的文件同步到服务器10.7.2.231上的/root/backup目录下
# 在10.7.2.230上执行以下命令：
rsync -avuz /root -e 'ssh -p10022' root@10.7.2.231:/root/backup
```

## 二、rsync安装

##### 1\. 源码安装

```handlebars
#（1）下载
wget https://download.samba.org/pub/rsync/src/rsync-3.2.3.tar.gz

#（2）解压并安装
tar -xvf rsync-3.2.3.tar.gz

#（3）编译安装
# 源码安装rsync时，其编译时所需要的gcc库文件尽量提前安装完毕
# 默认安装到/usr/local/目录下
./configure
make &&make install

#（4）设置开机启动
echo “/usr/local/bin/rsync --daemon -config=/etc/rsyncd.conf” >>/etc/profile
```

##### 2\. yum安装（centos中）

```handlebars
yum -y install rsync
```

## 三、参数说明

#### 1\. 常用参数

```handlebars
-v, --verbose详细模式输出

-a, --archive归档模式，表示以递归方式传输文件，并保持所有文件属性不变

-u, --update 仅仅进行更新，也就是跳过已经存在的目标位置，并且文件时间要晚于要备份的文件，不覆盖新的文件

-z,--compress对备份的文件在传输时进行压缩处理

--delete：删除那些DST中存在而在SRC中没有的文件

--exclude=PATTERN: 指定排除不需要传输的文件模式
```

#### 2\. 全部参数

```handlebars
-v, --verbose 详细模式输出
-q, --quiet 精简输出模式
-c, --checksum 打开校验开关，强制对文件传输进行校验
-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
-r, --recursive 对子目录以递归模式处理
-R, --relative 使用相对路径信息
# rsync foo/bar/foo.c remote:/tmp/      ## Rsync 参数在/tmp目录下创建foo.c文件，而如果使用-R参数：
# rsync -R foo/bar/foo.c remote:/tmp/   ## Rsync 参数会创建文件/tmp/foo/bar/foo.c，也就是会保持完全路径信息。
-b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
--backup-dir 将备份文件(如~filename)存放在在目录下。
-suffix=SUFFIX 定义备份文件前缀
-u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)
-l, --links 保留软链结
-L, --copy-links 想对待常规文件一样处理软链结
--copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结
--safe-links 忽略指向SRC路径目录树以外的链结
-k, --copy-dirlinks  transform symlink to a dir into referent dir
-K, --keep-dirlinks  treat symlinked dir on receiver as dir
-H, --hard-links 保留硬链结
-p, --perms 保持文件权限
-o, --owner 保持文件属主信息
-g, --group 保持文件属组信息
-D, --devices 保持设备文件信息
-t, --times 保持文件时间信息
-S, --sparse 对稀疏文件进行特殊处理以节省DST的空间
-n, --dry-run现实哪些文件将被传输
-W, --whole-file 拷贝文件，不进行增量检测
-x, --one-file-system 不要跨越文件系统边界
-B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节
-e, --rsh=COMMAND 指定替代rsh的shell程序
--rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息
-C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件
--existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件
--delete 删除那些DST中SRC没有的文件
--delete-excluded 同样删除接收端那些被该选项指定排除的文件
--delete-after 传输结束以后再删除
--ignore-errors 及时出现IO错误也进行删除
--max-delete=NUM 最多删除NUM个文件
--partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输
--force 强制删除目录，即使不为空
--numeric-ids 不将数字的用户和组ID匹配为用户名和组名
--timeout=TIME IP超时时间，单位为秒
-I, --ignore-times 不跳过那些有同样的时间和长度的文件
--size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间
--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0
-T --temp-dir=DIR 在DIR中创建临时文件
--compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份
-P 等同于 --partial
--progress 显示备份过程
-z, --compress 对备份的文件在传输时进行压缩处理
--exclude=PATTERN 指定排除不需要传输的文件模式
--include=PATTERN 指定不排除而需要传输的文件模式
--exclude-from=FILE 排除FILE中指定模式的文件
--include-from=FILE 不排除FILE指定模式匹配的文件
--version 打印版本信息
--address 绑定到特定的地址
--config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件
--port=PORT 指定其他的rsync服务端口
--blocking-io 对远程shell使用阻塞IO
-stats 给出某些文件的传输状态
--progress 在传输时现实传输过程
--log-format=FORMAT 指定日志文件格式
--password-file=FILE 从FILE中得到密码
--bwlimit=KBPS 限制I/O带宽，KBytes per second
-h, --help 显示帮助信息
```

## 四、rsync使用

由于ssh方式不需要进行配置文件配置，只需要像scp一样执行命令即可，因此本文主要介绍该方式的使用，rsync-daemon方式可参考其他教程。

例如需求为：将10.7.2.230上的/root目录文件同步到10.7.2.231上的/root/backup下，可通过以下两种方式来实现：

*   将本地文件推送（上传）到远端

```handlebars
# 在本地服务器上执行
# rsync  命令参数  源文件目录  目的目录
rsync [OPTION] SRC  [USER@]HOST:DEST

# 示例：在10.7.2.230上执行rsync，将/root目录文件同步到远程主机（10.7.2.231的/root/backup）上
rsync -avuz /root root@10.7.2.231:/root/backup
```

*   将远端文件拉（下载）到本地

```handlebars
# rsync  命令参数    源文件目录  目的目录
# rsync  [OPTION]  [USER@]HOST:SRC  [DEST]

# 示例：主机10.7.2.231上执行rsync，将10.7.2.230上/root下的目录文件下载到主机10.7.2.231的/root/backup上
rsync -auvz root@10.7.2.230:/root /root/backup  

#注意：
源目录加了斜线，效果就是将该目录下的内容传输到目标目录下
源目录不加斜线，效果就是将该目录传输到目标目录下
目标目录如果不存在，会自动创建目标目录
```

+ **本地拷贝目录，将/home/wwwroot/gigsgigs.com/拷贝到/data/wwwroot/gigsgigs.com/**

```shell
rsync -avu /home/wwwroot/gigsgigs.com/ /data/wwwroot/gigsgigs.com/
```

+ **将本地目录拷贝到远程服务器**

```shell
rsync -avu –progress –delete /home/wwwroot/gigsgigs.com/ root@1.2.3.4:/home/wwwroot/gigsgigs.com/

#备注
如果改了SSH端口，需要加-e “ssh -p 你的SSH端口”，如果不想显示具体传输过程可以去掉P参数，如果需要压缩传输可以加z参数。
–delete 参数，这样当本地删除的文件，远程端也会删除，保持完整的一致。
```

+ **将远程服务器目录拷贝到本地**

```shell
rsync -avu –progress –delete root@1.2.3.4:/home/wwwroot/gigsgigs.com/ /home/wwwroot/gigsgigs.com/
```

+ **可以用rsync快速删除大量文件**

```shell
建立一个空的文件夹： mkdir /root/blank
用rsync删除目标目录：rsync –delete-before -a -H -v –progress –stats /root/blank/ /home/wwwroot/cache/
这样我们要删除的 cache目录就会被清空了，删除的速 度会非常快。
```

+ **如果开启了iptables防火墙，请将873端口加入防火墙允许规则**

```shell
iptables -I INPUT -p tcp –dport 873 -j ACCEPT
iptables -I OUTPUT -p tcp –sport 873 -j ACCEPT
```

+ **用户可以自己根据自己的需求选择SSH或daemon模式**

```shell
配合crontab定时执行任务 自动完成同步、备份等工作。
```

```shell
将当前目录的hello.txt推送到服务端的backup模块
rsync -avz -P hello.txt  rsync_backup@192.168.253.153::backup --password-file=/etc/rsync_passwd 
或者
rsync -avz -P hello.txt  rsync://rsync_backup@192.168.253.153:/backup --password-file=/etc/rsync_passwd
 
将远端的backup目录拉取到当前目录
rsync -avz -P   rsync://rsync_backup@192.168.253.153:/backup ./  --password-file=/etc/rsync_passwd 
或者
rsync -avz -P   rsync_backup@192.168.253.153::backup ./  --password-file=/etc/rsync_passwd
也可以利用ssh
```





## 五、daemon模式的配置

rsync daemon模式是以rsync服务器形式运行，首先我们需要创建rsync服务器的配置文件，配置文件：/etc/rsyncd.conf 默认此文件可能不存在，需要自己创建，配置信息如下：

```shell
port = 873
uid = root
gid = root
use chroot = no //使用chroot到文件系统中的目录中
max connections = 100 //最大并发连接数

#motd file = /etc/rsyncd.motd //定义服务器信息的，自己写 rsyncd.motd 文件内容
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
log file = /var/log/rsyncd.log

timeout = 300

[gigsgigs] //自定义模块
path = /home/wwwroot/gigsgigs.com   //用来指定要备份的目录
ignore errors    //可以忽略一些IO错误
read only = no      //设置no，客户端可以上传文件，yes是只读
write only = no   //no为客户端可以下载，yes 不能下载
hosts allow = 192.168.2.0/24    //允许连接的IP，强烈建议加上你允许的IP，多个IP逗号隔开
list = yes   //客户请求时，使用模块列表
auth users = gigsgigs.com   //连接用户名，是虚拟用户与linux系统用户无关，多个用户名逗号隔开
secrets file = /etc/rsyncd.secrets   //验证密码文件，文件格式为：用户名:密码

写入配置时请将上面的注释信息去掉，并调整里面的相关参数。我们也提供了一个模板文件http://soft.vpser.net/sync/rsync/rsyncd.conf 可自己下载放到/etc/下并修改相关参数。

/etc/rsyncd.secrets 文件权限必须是600，创建好该文件后可以执行： chmod 600 /etc/rsyncd.secrets

注意：默认rsync服务器并不是自动启动的！

Debian/Ubuntu上是带自启动脚本的，修改 /etc/default/rsync ，将里面的RSYNC_ENABLE=false 改成 RSYNC_ENABLE=true 保存就设成开机自启动了。

CentOS上启动脚本都是不带的，执行：wget http://soft.vpser.net/sync/rsync/init.d.rsync -O /etc/init.d/rsync && chmod +x /etc/init.d/rsync && chkconfig rsync on 如果不报错的话就会开机自启动了。

完成上面设置后，执行：/etc/init.d/rsync start 即可启动。

测试rsync服务器：rsync -avuP gigsgigs.com@192.227.165.35::gigsgigs /home/wwwroot/gigsgigs.com/   进行连接测试，注：@前的vpser为自定义模块里设置的用户名，::后面的gigsgigs.com为你自定义模块的名称。
```

# rsync用法教程

**一、简介**

rsync 是一个常用的 Linux 应用程序，用于文件同步。

它可以在本地计算机与远程计算机之间，或者两个本地目录之间同步文件（但不支持两台远程计算机之间的同步）。它也可以当作文件复制工具，替代`cp`和`mv`命令。

它名称里面的`r`指的是 remote，rsync 其实就是”远程同步”（remote sync）的意思。与其他文件传输工具（如 FTP 或 scp）不同，rsync 的最大特点是会检查发送方和接收方已有的文件，仅传输有变动的部分（默认规则是文件大小或修改时间有变动）。

**二、安装**

如果本机或者远程计算机没有安装 rsync，可以用下面的命令安装。

```
# Debian
$ sudo apt-get install rsync

# Red Hat
$ sudo yum install rsync

# Arch Linux
$ sudo pacman -S rsync
```

注意，传输的双方都必须安装 rsync。

**三、基本用法**

本机使用 rsync 命令时，可以作为`cp`和`mv`命令的替代方法，将源目录同步到目标目录。

```
$ rsync -r source/ destination
```

上面命令中，`-r`表示递归，即包含子目录。注意，`-r`是必须的，否则 rsync 运行不会成功。`source`目录表示源目录，`destination`表示目标目录。

如果有多个文件或目录需要同步，可以写成下面这样。

```
$ rsync -r source1 source2 destination
```

上面命令中，`source1`，`source2`都会被同步到`destination`目录。

**3.2 `-a` 参数**

`-a`参数可以替代`-r`，除了可以递归同步以外，还可以同步元信息（比如修改时间、权限等）。由于 rsync 默认使用文件大小和修改时间决定文件是否需要更新，所以`-a`比`-r`更有用。下面的用法才是常见的写法。

```
$ rsync -a source destination
```

目标目录`destination`如果不存在，rsync 会自动创建。执行上面的命令后，源目录`source`被完整地复制到了目标目录`destination`下面，即形成了`destination/source`的目录结构。

如果只想同步源目录`source`里面的内容到目标目录`destination`，则需要在源目录后面加上斜杠。

```
$ rsync -a source/ destination
```

上面命令执行后，`source`目录里面的内容，就都被复制到了`destination`目录里面，并不会在`destination`下面创建一个`source`子目录。

### **3.3 `-n` 参数**

如果不确定 rsync 执行后会产生什么结果，可以先用`-n`或`--dry-run`参数模拟执行的结果。

```
$ rsync -anv source/ destination
```

上面命令中，`-n`参数模拟命令执行的结果，并不真的执行命令。`-v`参数则是将结果输出到终端，这样就可以看到哪些内容会被同步。

### **3.4 `–delete` 参数**

默认情况下，rsync 只确保源目录的所有内容（明确排除的文件除外）都复制到目标目录。它不会使两个目录保持相同，并且不会删除文件。如果要使得目标目录成为源目录的镜像副本，则必须使用`--delete`参数，这将删除只存在于目标目录、不存在于源目录的文件。

```
$ rsync -av --delete source/ destination
```

上面命令中，`--delete`参数会使得`destination`成为`source`的一个镜像。

## **四、排除文件**

### **4.1 `–exclude` 参数**

有时，我们希望同步时排除某些文件或目录，这时可以用`--exclude`参数指定排除模式。

```
$ rsync -av --exclude='*.txt' source/ destination
# 或者
$ rsync -av --exclude '*.txt' source/ destination
```

上面命令排除了所有 TXT 文件。

注意，rsync 会同步以”点”开头的隐藏文件，如果要排除隐藏文件，可以这样写`--exclude=".*"`。

如果要排除某个目录里面的所有文件，但不希望排除目录本身，可以写成下面这样。

```
$ rsync -av --exclude 'dir1/*' source/ destination
```

多个排除模式，可以用多个`--exclude`参数。

```
$ rsync -av --exclude 'file1.txt' --exclude 'dir1/*' source/ destination
```

多个排除模式也可以利用 Bash 的大扩号的扩展功能，只用一个`--exclude`参数。

```
$ rsync -av --exclude={'file1.txt','dir1/*'} source/ destination
```

如果排除模式很多，可以将它们写入一个文件，每个模式一行，然后用`--exclude-from`参数指定这个文件。

```
$ rsync -av --exclude-from='exclude-file.txt' source/ destination
```

### **4.2 `–include` 参数**

`--include`参数用来指定必须同步的文件模式，往往与`--exclude`结合使用。

```
$ rsync -av --include="*.txt" --exclude='*' source/ destination
```

上面命令指定同步时，排除所有文件，但是会包括 TXT 文件。

## **五、远程同步**

### **5.1 SSH 协议**

rsync 除了支持本地两个目录之间的同步，也支持远程同步。它可以将本地内容，同步到远程服务器。

```
$ rsync -av source/ username@remote_host:destination
```

也可以将远程内容同步到本地。

```
$ rsync -av username@remote_host:source/ destination
```

如果 ssh 命令有附加的参数，则必须使用`-e`参数指定所要执行的 SSH 命令。

```
# 设置端口为22，设置对应的pem等
$ sync -av -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /home/xiaojin/Desktop/temp/ root@8.134.210.33:/root/temp/temp_rsync
```

### **5.2 rsync 协议（这个我没验证）**

除了使用 SSH，如果另一台服务器安装并运行了 rsync 守护程序，则也可以用`rsync://`协议（默认端口873）进行传输。具体写法是服务器与目标目录之间使用双冒号分隔`::`。

```
$ rsync -av source/ 192.168.122.32::module/destination
```

注意，上面地址中的`module`并不是实际路径名，而是 rsync 守护程序指定的一个资源名，由管理员分配。

如果想知道 rsync 守护程序分配的所有 module 列表，可以执行下面命令。

```
$ rsync rsync://192.168.122.32
```

rsync 协议除了使用双冒号，也可以直接用`rsync://`协议指定地址。

```
$ rsync -av source/ rsync://192.168.122.32/module/destination
```

## **六、增量备份（这个我没验证）**

rsync 的最大特点就是它可以完成增量备份，也就是默认只复制有变动的文件。

除了源目录与目标目录直接比较，rsync 还支持使用基准目录，即将源目录与基准目录之间变动的部分，同步到目标目录。

具体做法是，第一次同步是全量备份，所有文件在基准目录里面同步一份。以后每一次同步都是增量备份，只同步源目录与基准目录之间有变动的部分，将这部分保存在一个新的目标目录。这个新的目标目录之中，也是包含所有文件，但实际上，只有那些变动过的文件是存在于该目录，其他没有变动的文件都是指向基准目录文件的硬链接。

`--link-dest`参数用来指定同步时的基准目录。

```
$ rsync -a --delete --link-dest /compare/path /source/path /target/path
```

上面命令中，`--link-dest`参数指定基准目录`/compare/path`，然后源目录`/source/path`跟基准目录进行比较，找出变动的文件，将它们拷贝到目标目录`/target/path`。那些没变动的文件则会生成硬链接。这个命令的第一次备份时是全量备份，后面就都是增量备份了。

下面是一个脚本示例，备份用户的主目录。

```
#!/bin/bash

# A script to perform incremental backups using rsync

set -o errexit
set -o nounset
set -o pipefail

readonly SOURCE_DIR="${HOME}"
readonly BACKUP_DIR="/mnt/data/backups"
readonly DATETIME="$(date '+%Y-%m-%d_%H:%M:%S')"
readonly BACKUP_PATH="${BACKUP_DIR}/${DATETIME}"
readonly LATEST_LINK="${BACKUP_DIR}/latest"

mkdir -p "${BACKUP_DIR}"

rsync -av --delete \
  "${SOURCE_DIR}/" \
  --link-dest "${LATEST_LINK}" \
  --exclude=".cache" \
  "${BACKUP_PATH}"

rm -rf "${LATEST_LINK}"
ln -s "${BACKUP_PATH}" "${LATEST_LINK}"
```

上面脚本中，每一次同步都会生成一个新目录`${BACKUP_DIR}/${DATETIME}`，并将软链接`${BACKUP_DIR}/latest`指向这个目录。下一次备份时，就将`${BACKUP_DIR}/latest`作为基准目录，生成新的备份目录。最后，再将软链接`${BACKUP_DIR}/latest`指向新的备份目录。

## **七、配置项**

`-a`、`--archive`参数表示存档模式，保存所有的元数据，比如修改时间（modification time）、权限、所有者等，并且软链接也会同步过去。

`--append`参数指定文件接着上次中断的地方，继续传输。

`--append-verify`参数跟`--append`参数类似，但会对传输完成后的文件进行一次校验。如果校验失败，将重新发送整个文件。

`-b`、`--backup`参数指定在删除或更新目标目录已经存在的文件时，将该文件更名后进行备份，默认行为是删除。更名规则是添加由`--suffix`参数指定的文件后缀名，默认是`~`。

`--backup-dir`参数指定文件备份时存放的目录，比如`--backup-dir=/path/to/backups`。

`--bwlimit`参数指定带宽限制，默认单位是 KB/s，比如`--bwlimit=100`。

`-c`、`--checksum`参数改变`rsync`的校验方式。默认情况下，rsync 只检查文件的大小和最后修改日期是否发生变化，如果发生变化，就重新传输；使用这个参数以后，则通过判断文件内容的校验和，决定是否重新传输。

`--delete`参数删除只存在于目标目录、不存在于源目标的文件，即保证目标目录是源目标的镜像。

`-e`参数指定使用 SSH 协议传输数据。

`--exclude`参数指定排除不进行同步的文件，比如`--exclude="*.iso"`。

`--exclude-from`参数指定一个本地文件，里面是需要排除的文件模式，每个模式一行。

`--existing`、`--ignore-non-existing`参数表示不同步目标目录中不存在的文件和目录。

`-h`参数表示以人类可读的格式输出。

`-h`、`--help`参数返回帮助信息。

`-i`参数表示输出源目录与目标目录之间文件差异的详细情况。

`--ignore-existing`参数表示只要该文件在目标目录中已经存在，就跳过去，不再同步这些文件。

`--include`参数指定同步时要包括的文件，一般与`--exclude`结合使用。

`--link-dest`参数指定增量备份的基准目录。

`-m`参数指定不同步空目录。

`--max-size`参数设置传输的最大文件的大小限制，比如不超过200KB（`--max-size='200k'`）。

`--min-size`参数设置传输的最小文件的大小限制，比如不小于10KB（`--min-size=10k`）。

`-n`参数或`--dry-run`参数模拟将要执行的操作，而并不真的执行。配合`-v`参数使用，可以看到哪些内容会被同步过去。

`-P`参数是`--progress`和`--partial`这两个参数的结合。

`--partial`参数允许恢复中断的传输。不使用该参数时，`rsync`会删除传输到一半被打断的文件；使用该参数后，传输到一半的文件也会同步到目标目录，下次同步时再恢复中断的传输。一般需要与`--append`或`--append-verify`配合使用。

`--partial-dir`参数指定将传输到一半的文件保存到一个临时目录，比如`--partial-dir=.rsync-partial`。一般需要与`--append`或`--append-verify`配合使用。

`--progress`参数表示显示进展。

`-r`参数表示递归，即包含子目录。

`--remove-source-files`参数表示传输成功后，删除发送方的文件。

`--size-only`参数表示只同步大小有变化的文件，不考虑文件修改时间的差异。

`--suffix`参数指定文件名备份时，对文件名添加的后缀，默认是`~`。

`-u`、`--update`参数表示同步时跳过目标目录中修改时间更新的文件，即不同步这些有更新的时间戳的文件。

`-v`参数表示输出细节。`-vv`表示输出更详细的信息，`-vvv`表示输出最详细的信息。

`--version`参数返回 rsync 的版本。

`-z`参数指定同步时压缩数据。

## **八、套娃现象**

`source`不带斜杆和带斜杆的区别是，一个是传文件夹过去，一个是传文件夹中的文件过去。

```
$ rsync -a source/ destination
```

不会套娃：

```
sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest/ ubuntu@69.230.236.43:/ldata/temp2/synctest

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest/ ubuntu@69.230.236.43:/ldata/temp2/synctest/
```

如果文件已存在，会出现套娃

```
sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest ubuntu@69.230.236.43:/ldata/temp2/synctest

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest ubuntu@69.230.236.43:/ldata/temp2/synctest/

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/ ubuntu@69.230.236.43:/ldata/temp2/synctest

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/ ubuntu@69.230.236.43:/ldata/temp2/synctest/
```

那个套两层路径，可能是这个写法

```
sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp ubuntu@69.230.236.43:/ldata/temp2/synctest/

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp ubuntu@69.230.236.43:/ldata/temp2/synctest
```

不会套娃

```
sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest ubuntu@69.230.236.43:/ldata/temp2

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest ubuntu@69.230.236.43:/ldata/temp2/
```

不会套娃，但会传到上一层目录

```
sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest/ ubuntu@69.230.236.43:/ldata/temp2

sudo rsync -auv -e 'ssh -p 22 -i /home/xiaojing/MARS.pem' /ldata/temp/synctest/ ubuntu@69.230.236.43:/ldata/temp2/
```

**实例、可以配合宝塔面板的计划任务间隔几分钟或几小时或者几天自动执行同步**

```
rsync -aWv --delete /www/wwwroot/ /home/www/backup/wwwroot
# 同步/www/wwwroot目录下面的文件夹及其子目录和文件到/home/www/backup/wwwroot目录并保持文件夹结构及文件属性和权限 
# -a表示保持文件原属性 
# -W表示直接同步不校验文件，否则大文件很费时间 
# -v表示输出同步信息 
# --delete表示如果原目录有删除文件那么目标目录页删除，就能保持目标目录是原目录的一个镜像文件夹

rsync -aWv --delete /www/backup/ /home/www/backup/backup
# 同步/www/wwwroot目录下面的文件夹及其子目录和文件到/home/www/backup/wwwroot目录并保持文件夹结构及文件属性和权限 
# -a表示保持文件原属性 
# -W表示直接同步不校验文件，否则大文件很费时间 
# -v表示输出同步信息 
# --delete表示如果原目录有删除文件那么目标目录页删除，就能保持目标目录是原目录的一个镜像文件夹
```

**rsync怎么通过ssh把本地服务器文件夹同步到远程服务器**

通过SSH来安全地同步本地服务器上的文件夹到远程服务器。基本的命令结构如下：

```
rsync -avz -e "ssh -p SSH_PORT" LOCAL_DIRECTORY USER@REMOTE_HOST:REMOTE_DIRECTORY
```

这里是各部分参数的解释：

- `-a` 参数表示归档模式，它会保持原始文件的权限、时间戳、软硬链接等。
- `-v` 参数表示详细模式，会显示同步的详细信息。
- `-z` 参数表示启用压缩，它会在传输时将数据压缩，以减少网络上的数据量。
- `-e` 参数用于指定使用的shell，这里指定了`ssh`，后面可以跟`ssh`的相关参数，比如端口。
- `SSH_PORT` 是SSH服务器监听的端口，如果是默认的22端口，可以省略`-p SSH_PORT`部分。
- `LOCAL_DIRECTORY` 是你想要同步的本地服务器上的文件夹路径。
- `USER` 是你在远程服务器上的用户名。
- `REMOTE_HOST` 是远程服务器的地址。
- `REMOTE_DIRECTORY` 是远程服务器上你想要同步到的目标文件夹路径。

举个例子，如果你要将本地的`/var/www`文件夹同步到远程服务器`192.168.1.10`上用户`username`的主目录下的`www`文件夹，并且远程服务器SSH端口是2222，你可以这样做：

```
rsync -avz -e "ssh -p 2222" /var/www/ username@192.168.1.10:~/www/
```

请注意，如果本地服务器或远程服务器使用的不是默认的SSH密钥，你可能需要使用`-i`参数来指定私钥文件的路径。

```
rsync -avz -e "ssh -p SSH_PORT -i /path/to/private_key" LOCAL_DIRECTORY USER@REMOTE_HOST:REMOTE_DIRECTORY
```

确保在使用`rsync`之前，本地服务器和远程服务器之间的SSH免密钥登录已经设置好，这样`rsync`在同步时不会要求你输入密码。



**如果远程服务器是使用的用户名和密码怎么同步**

如果远程服务器需要用户名和密码进行身份验证，而不是基于SSH密钥的认证，您通常需要在`rsync`命令执行时输入密码。默认情况下，`rsync`会提示您输入密码。但是，这种方式不适合自动化脚本，因为它需要交互式输入。

对于自动化脚本，您可以使用`sshpass`工具来自动输入密码。首先，确保您的系统中已经安装了`sshpass`。

安装`sshpass`的命令通常是：

```
# 在Debian/Ubuntu系统上
sudo apt-get install sshpass

# 在RedHat/CentOS系统上
sudo yum install sshpass

# 在Fedora系统上
sudo dnf install sshpass
```

然后，使用`sshpass`结合`rsync`来同步文件夹：

```
sshpass -p 'YOUR_PASSWORD' rsync -avz -e "ssh -o StrictHostKeyChecking=no" LOCAL_DIRECTORY USER@REMOTE_HOST:REMOTE_DIRECTORY
```

这里是各部分参数的解释：

- `sshpass -p 'YOUR_PASSWORD'` 是`sshpass`命令，用于自动提供密码。替换`YOUR_PASSWORD`为您的实际密码。
- `-o StrictHostKeyChecking=no` 参数是为了避免手动验证远程主机的密钥，这在自动化脚本中很有用。

请注意，使用`sshpass`来自动输入密码可能会带来安全风险，因为密码是以明文形式在命令行中显示。确保在信任的环境中使用这种方法，并对密码的安全性负责。

最后，如果您不是自动化脚本的环境，建议使用SSH密钥认证方式进行`rsync`同步，因为这种方式更为安全。如果您没有选择，只能使用用户名和密码，请尽量在命令行中直接输入密码，而不是将密码放在脚本或命令中。



**rsync同步能显示进度吗**

是的，`rsync`可以显示同步的进度。要显示单个文件的进度以及整体的传输统计信息，您可以使用`--progress`或`-P`标志（这也包括`--partial`的功能，允许部分传输的文件在中断后继续传输）。此外，使用`-h`标志可以使输出更易于阅读，因为它会提供人类可读的格式。

下面是一个示例命令，它将显示进度信息：

```
rsync -avzhP -e "ssh -p SSH_PORT" LOCAL_DIRECTORY USER@REMOTE_HOST:REMOTE_DIRECTORY
```

在这个命令中：

- `-a` 是归档模式，它递归并保持几乎所有的文件属性。
- `-v` 是详细模式，显示更多信息。
- `-z` 启用压缩以减少网络传输数据量。
- `-h` 使输出更易于阅读。
- `-P` 相当于`--partial --progress`，它允许部分传输和显示进度。
- `-e` 指定使用的shell，后面跟SSH具体参数。

当您使用`-P`或`--progress`时，`rsync`会在传输每个文件时显示一个进度条，包括已传输的百分比、传输速度和预计剩余时间。

请注意，如果您正在传输很多小文件，单个文件的进度显示可能会过于快速，从而难以阅读。在这种情况下，您仍然可以获得总体传输的统计信息。

## 例如使用SSH同步宝塔面板的网站和备份数据库到远程服务器例子

```
备份网站
rsync -aWvhP --delete -e "ssh -p 22" /www/wwwroot/ root@192.168.1.1:/www/ycbackupwwwroot/wwwroot #/www/wwwroot/是本地目录 /www/ycbackupwwwroot/wwwroot是远程目录

备份数据库备份文件
rsync -aWvhP --delete -e "ssh -p 22" /www/backup/ root@192.168.1.1:/www/ycbackupwwwroot/backup #/www/backup/是本地目录 /www/ycbackupwwwroot/backup是远程目录

提示需要输入远程服务器root用户的密码就输入就行，如果想让自动输入密码就按照上面的教程使用sshpass工具来自动输入密码

同步期间终端如果关闭会停止同步，想让关闭终端不停止同步请看
https://www.sanradar.com/archives/566
此篇文章让rsync在后台运行
```

**将远程服务器文件同步到本地例子**

```
sshpass -p '123456' rsync -aWvhP --delete -e "ssh -p 22 -o StrictHostKeyChecking=no" root@192.168.1.1:/www/backup/ /www/wwwroot/ycbackupwwwroot/backup
表示使用ssh连接，端口号是22，自动输入远程服务器192.168.1.1的root用户的ssh密码123456， 将远程服务器的/www/backup/目录 同步到本地的/www/wwwroot/ycbackupwwwroot/backup目录
```

#### rsync断点续传

```shell
rsync -avz --partial --append-verify --progress -e "ssh -p 8889" /home/SaasProject/file/* root@14.23.49.154:/home/data/SaasProject/file/

```

> - **`--partial`**：如果传输中断，此选项将保留部分传输的文件。否则，`rsync` 将删除未完成的文件，并从头开始传输。
> - **`--append`**：重新启动时，`rsync` 将数据追加到部分传输的文件中，而不是重新开始。
> - **`--progress`**：显示传输进度，特别适合于传输大文件时使用。

### 注意事项：

- **网络中断**：如果网络连接不稳定，使用这些选项的 `rsync` 将能够优雅地处理中断，并在连接恢复后继续传输。
- **文件完整性**：`--partial` 和 `--append` 组合可以确保文件正确续传，但您也可以添加 `--append-verify` 选项，以在传输后验证文件完整性：