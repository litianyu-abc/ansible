# centos7部署杀毒软件clamav

## 1.下载安装

> [clamav](https://www.clamav.net/downloads) 官网下载 [clamav- 1.0.1.linux.x86_64.rpm](https://www.clamav.net/downloads/production/clamav-1.0.1.linux.x86_64.rpm)

```shell
# 有网络可用如下命令下载
wget https://www.clamav.net/downloads/production/clamav-1.0.1.linux.x86_64.rpm
```

![img](https://blog.ncyupu.com/wp-content/uploads/2023/06/8b008b09bfc1ed9-1024x424.png)

> 将该文件上传至服务器，安装命令如下：

```shell
rpm -ivh --prefix=/usr/local/clamav clamav-1.0.1.linux.x86_64.rpm
```

## 2. 配置

```shell
添加用户组和组成员
groupadd clamav
useradd -g clamav clamav
创建日志目录、病毒库目录和套接字目录
mkdir -p /usr/local/clamav/logs
mkdir -p /usr/local/clamav/update
mkdir -p /usr/local/clamav/socket
创建日志文件
touch /usr/local/clamav/logs/clamd.log
touch /usr/local/clamav/logs/freshclam.log
文件授权
chown clamav:clamav /usr/local/clamav/logs/clamd.log
chown clamav:clamav /usr/local/clamav/logs/freshclam.log
chown clamav:clamav /usr/local/clamav/logs
chown clamav:clamav /usr/local/clamav/update
chown clamav:clamav /usr/local/clamav/socket
修改配置文件
cp /usr/local/clamav/etc/clamd.conf.sample /usr/local/clamav/etc/clamd.conf
cp /usr/local/clamav/etc/freshclam.conf.sample /usr/local/clamav/etc/freshclam.conf
文件1：clamd.conf
vim /usr/local/clamav/etc/clamd.conf
#Example　　//注释掉这一行
#添加以下内容 LogFile /usr/local/clamav/logs/clamd.log
PidFile /usr/local/clamav/update/clamd.pid
DatabaseDirectory /usr/local/clamav/update
LocalSocket /usr/local/clamav/socket/clamd.socke
文件2：freshclam.conf
vim /usr/local/clamav/etc/freshclam.conf
#Example　　//注释掉这一行
#添加以下内容
DatabaseDirectory /usr/local/clamav/update
UpdateLogFile /usr/local/clamav/logs/freshclam.log
PidFile /usr/local/clamav/update/freshclam.pid
将这两个文件复制一下：cp /usr/local/clamav/etc/*.conf /usr/local/etc/
```

## 3. 运行

```shell
配置库文件路径
vim /etc/ld.so.conf
追加一行：/usr/local/clamav/lib64
更新生效：ldconfig
如果最后运行时仍然报错：clamscan: error while loading shared libraries: libclamav.so.9: cannot open shared object file: No such file or directory
则说明配置没有生效。
下载病毒库文件并上传到目录 /usr/local/clamav/update
main.cvd
daily.cvd
bytecode.cvd
注：也可以在有网络的机器上运行如下命令更新病毒库：/usr/local/clamav/bin/freshclam
创建命令软件链接
ln -s /usr/local/clamav/bin/clamscan /usr/local/bin/clamscan
ln -s /usr/local/clamav/bin/freshclam /usr/local/bin/freshclam
运行使用
# clamscan -r 指定目录（不填则默认当前目录）
clamscan -r
# 后台运行
nohup clamscan -r / > clamscanNohup.log 2>&1 &
```

![image-20240809090044321](C:\Users\EDY\AppData\Roaming\Typora\typora-user-images\image-20240809090044321.png)

1. 卸载程序

   - `rpm remove clama`

   ## 4.服务器配置

   ### 4.1 设置 daemon [守护进程](https://so.csdn.net/so/search?q=守护进程&spm=1001.2101.3001.7020)（推荐）

   1. 开机自动更新病毒库

      - `*# 启动clamav守护进程*`
      - `freshclam --daemon`
      - `*# 设置freshclam开机自启动*`
      - `echo "/usr/local/clamav/bin/freshclam --daemon" >> /etc/rc.d/rc.local`

   2. 守护模式启动程序

      - `/usr/local/clamav/sbin/clamd`

   3. 检查是否开启守护进程

      - `*# 运行如下命令，其中 TPGID 显示为 -1* ps ajx | more`
      - *ps ajx | grep freshclam | ps ajx | grep clamd*

      ![img](https://blog.ncyupu.com/wp-content/uploads/2023/06/78805a221a988e7-20.png)

   ![img](https://blog.ncyupu.com/wp-content/uploads/2023/06/78805a221a988e7-21.png)

### 4.2 定时任务

配置在定时任务中：

```shell
# 打开定时任务配置文件
crontab -e
# 升级病毒库
1 2  * * *	/usr/local/clamav/bin/freshclam
# 定时查杀指定目录并删除感染的文件
1 3 * * *	clamscan -r / --remove -l /var/log/clamscan.log
```

contab命令说明

```shell
在 crontab 文件中，通过 m h dom mon dow command 这六个字段来设置定时任务，每一行对应一个定时任务。这六个字段的含义说明如下：

m：对应分钟（minute）
指定要在一小时之中的第几分钟执行该任务。取值范围是 0-59.
h：对应小时（hour）
指定要在一天之中的第几个小时执行该任务。取值范围是 0-23.
dom：对应日期（day of month）
指定要在一月之中的第几天执行该任务。取值范围是 0-31.
mon：对应月份（month）
指定要在一年之中的第几月执行该任务。取值范围是 1-12。
也可以通过月份英文名称的前三个字母来指定，不区分大小写。例 如，一月的英文单词是 january，那么这里可以用 jan 来指定一月。
dow：对应星期几（day of week）
指定要在一周之中的星期几执行该任务。取值范围是 0-7，0 和 7 都对应星期天。
也可以通过星期英文名称的前三个字母来指定，不区分大小写。例如，星期一的英文单词是 monday，那么这里可以用 mon 来指定星期一。
command：对应具体的操作提供具体的命令来指定进行什么操作，可以提供脚本文件的路径来执行该脚本文件。这六个字段要求用空格隔开。且每个字段都必须提供值，不能省略某个字段的值。从第五个字段之后的所有内容都属于第六个字段，也就是要执行的操作。
前五个字段可以使用下面的特殊字符来指定一些特殊的时间：

表示任意一个有效的取值。例如，把日期指定为 *，则表示每一天都进行该任务。
-表示一个有效的范围值。例如，在小时指定为 8-11，表示在 8点、9点、10点、和 11点都执行该任务。
,表示隔开不同的取值列表。例如，把小时指定为 2,3,5,7，表示在 2点、3点、5点、7点都执行该任务。注意：在逗号后面不要加空格，空格表示隔开不同的字段。
/表示一个时间间隔，而不是指定具体的时间。例如，把小时指定为 */2，表示每间隔两小时执行一次该任务。
```

## 5.[ClamAV](https://so.csdn.net/so/search?q=ClamAV&spm=1001.2101.3001.7020) 常用命令

```shell
clamscan  -h
freshclam -h
```

> clamscan:

```shell
通用，不依赖服务，命令参数较多，执行速度稍慢；
用clamscan扫描，不需要开始服务就能使用；
-r 递归扫描子目录
-i 只显示发现的病毒文件
--no-summary 不显示统计信息 

扫描参数：
-r/--recursive[=yes/no]               所有文件
--log=FILE/-l FILE                    增加扫描报告
--move [路径]        		      移动病毒文件至..
--remove [路径]      	              删除病毒文件
--quiet               		      只输出错误消息
--infected/-i         		      只输出感染文件
--suppress-ok-results/-o              跳过扫描OK的文件
--bell                      	      扫描到病毒文件发出警报声音
--unzip(unrar)                        解压压缩文件扫描
```



## ubuntu安装部署ClamAV

> ClamAV 简介 ClamAV 是一个开源的防病毒软件，可用于检测木马，病毒，恶意软件和其他恶意威胁，适用于 Linux、macOS 和 Windows 平台。
>
> 官网网站：http://www.clamav.net/downloads

**Ubuntu Linux平台安装方法如下：**

```SHELL
    sudo  apt  update

    sudo  apt  install  clamav  

    安装完成后，首次使用需要手动更新病毒库。手动更新病毒库之前，需要将停止clamav守护进程，否则会报错，无法更新病毒库。需要输入以下命令：

    sudo  service  clamav-freshclam  stop      #停止守护进程

    然后，手动更新病毒库，输入以下命令：

    sudo  freshclam

    首次，更新病毒库需要较长时间。更新完毕后，需要重启守护进程，使系统以后自动更新病毒库。输入以下命令：          

    sudo  service  clamav-freshclam  restart

    # 显示当前病毒库的版本

    sudo freshclam -V

    完成更新病毒库后，就可以采用clamav扫描某个目录，检测病毒了。可以输入以下命令：      

    clamscan  -r    目录        

    其中，-r参数，表示递归查找文件。clamscan命令完成检测病毒后，会生成一个报告。

使用帮助如下：

    clamscan –ri / -l clamscan.log --remove 这里递归扫描根目录 / ，发现感染文件立即删除

    -r 递归扫面子文件

    –i 只显示被感染的文件

    -l 指定日志文件

    --remove 删除被感染文件

    --move 隔离被感染文件

定时更新和查杀

   # 导出当前crontab到临时文件crontab.conf
   crontab -l > crontab.conf
   # 向临时文件追加计划任务
   echo -ne '
   0 1 * * *  /usr/local/bin/freshclam --quiet
   0 2 * * *  /usr/local/bin/clamscan -vri /root/ --move=/usr/local/clamav/infected -l    /usr/local/clamav/logs/clamscan-$(date +\%Y\%m\%d).log
   ' >> crontab.conf
   # 引用文件导入crontab
   crontab crontab.conf
   # 重启crond服务
   systemctl restart crond.service
   # 删除临时文件
   rm -f crontab.conf
```

![img](https://yan-jian.com/file/20240122/1.jpg)
