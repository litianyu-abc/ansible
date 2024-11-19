**前言**

看过上一篇接手老项目的痛——[MongoDB](https://cloud.tencent.com/product/mongodb?from_column=20065&from=20065)学习及集群搭建知道，最近接手了一个后妈养的项目，项目的[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)没有人维护，DBA以各种理由推脱暂时不接，面对裸奔没有备份的数据库，我的内心很焦灼，于是花了点时间把生产环境的自动备份给搞起来。

下面话不多说了，来一起看看详细的介绍吧

**一些准备**

既然都备份了，为了保险起见，备份与库就不放在同一台[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)上了，于是向运维申请了一台服务器，同时安装好mongo，如果不知道怎么安装mongo的话可以看我的上一篇文章。

安装完之后，首先测试下是否可以远程访问目标mongodb,到安装好mongo的bin目录下

代码语言：javascript

复制

```javascript
./mongo 10.100.1.101:27017 #目标mongo的ip及端口
```

然后创建些必要的目录，比如备份文件放在哪个目录之类的。

接下来测试下利用mongodump来备份数据库：

代码语言：javascript

复制

```javascript
./bin/mongodump --host test/10.100.1.101:27017,10.100.1.102:27017 -d testdb --out /data/temp

# test为副本集名称
# 10.100.1.101:27017,10.100.1.102:27017为副本集节点，有多个可以多个
# -d testdb是要备份的库名，不填默认副本集下全部
# --out 保存路径
```

到这里，mongo的备份已经实现，现在要完成的就是自动啦。

**编写脚本**

自动定时备份其实就是通过crontab命令来实现啦。但前提是我们需要编写个定时跑的脚本。首先我们新建个脚本：

代码语言：javascript

复制

```javascript
vi /home/local/mongod_bak.sh
```

然后编写对应的脚本，脚本上有对应的注释，供大家参考，这里主要做了三个动作，首先是备份，然后将备份的文件压缩，然后只保留最近7天的文件。

代码语言：javascript

复制

```javascript
#!/bin/bash
sourcepath='/home/local/mongodb/bin'  #mongodb文件路径
targetpath='/home/local/mongodb_bak' #备份的路径
nowtime=$(date +%Y-%m-%d-%H)
replicationname='test'  #副本集名
dbname='testdb' #库名
port='27017' #端口
ip1='10.100.1.101' #ip
ip2='10.100.1.102'

echo "============== start backup ${nowtime} =============="
start()
{
 ${sourcepath}/mongodump --host ${replicationname}/${ip1}:${port},${ip2}:${port} -d ${dbname} --out ${targetpath}/${nowtime}
}
execute()
{
 start
 if [ $? -eq 0 ]
 then
 echo "back successfully!"
 else
 echo "back failure!"
 fi
}
 
if [ ! -d "${targetpath}/${nowtime}/" ]
then
 mkdir ${targetpath}/${nowtime}
fi
execute
echo "============== back end ${nowtime} =============="

echo "============== start zip ${nowtime} =============="
zip -r ${targetpath}/${nowtime}.zip ${targetpath}/${nowtime}
rm -rf ${targetpath}/${nowtime}
echo "============== zip end ${nowtime} =============="

echo "============== start delete seven days ago back ${nowtime} =============="
find ${targetpath} -type f -mtime +7 -name "*" -exec rm -rf {} \; 
echo "============== delete end ${nowtime} =============="
```

编写完之后，给到文件可执行权限，并可以手动执行测试下：

代码语言：javascript

复制

```javascript
chmod +x /home/local/mongod_bak.sh
```

**定时任务**

最后就是添加执行计划了，修改/etc/crontab

代码语言：javascript

复制

```javascript
crontab -e
```

添加执行脚本,保存即可。

代码语言：javascript

复制

```javascript
30 1 * * * /home/local/mongod_bak.sh #表示每天凌晨1点30执行备份
```

这里简单介绍下crontab。

crontab命令常见于Unix和类Unix的操作系统之中，用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于crontab文件中，以供之后读取和执行。

通常，crontab储存的指令被守护进程激活， crond常常在后台运行，每一分钟检查是否有预定的作业需要执行。这类作业一般称为cron jobs。

一些常用命令可以参考下：

代码语言：javascript

复制

```javascript
#启动服务
/sbin/service crond start 

#关闭服务
/sbin/service crond stop 

#重启服务
/sbin/service crond restart 

#重新载入配置
/sbin/service crond reload 

#查看crontab服务状态
service crond status 

#手动启动crontab服务
service crond start 

#查看crontab服务是否已设置为开机启动，执行命令：
ntsysv

#加入开机自动启动:
chkconfig --level 35 crond on

#列出crontab文件
crontab -l

#编辑crontab文件
crontab -e

#删除crontab文件
$ crontab -r

#恢复丢失的crontab文件
#假设你在自己的$HOME目录下还有一个备份，那么可以将其拷贝到/var/spool/cron/<username>，其中<username >是用户名
#或者使用如下命令其中，<filename>是你在$HOME目录中副本的文件名
crontab <filename>
```

**总结**

慢工出细活，有些东西一开始觉得很难很麻烦，但当你静下心来认真研究下，还是很容易理解的，毕竟你不是第一个踩坑的，所以还是好好学习吧。

好了，以上就是这篇文章的全部内容了，希望本文的内容对大家的学习或者工作具有一定的参考学习价值，如果有疑问大家可以留言交流，谢谢大家对ZaLou.Cn的支持。

本文转自 <https://cloud.tencent.com/developer/article/1721781?from=15425>，如有侵权，请联系删除。