Linux,数据库,配置相关视频讲解：

 [用vim复制粘贴\_保持双手正位](https://edu.51cto.com/video/4782.html?utm_platform=pc&utm_medium=51cto&utm_source=zhuzhan&utm_content=wzy_tl)

 [用python编程Excel有没有用处？](https://edu.51cto.com/video/4470.html?utm_platform=pc&utm_medium=51cto&utm_source=zhuzhan&utm_content=wzy_tl)

 [013为什么说未来系统是linux](https://edu.51cto.com/video/4392.html?utm_platform=pc&utm_medium=51cto&utm_source=zhuzhan&utm_content=wzy_tl)

  

  

### 目录

*   1\. 下载安装包
*   2\. 将安装包上传至服务器
*   3\. 解压
*   4\. 创建配置文件
*   5\. 修改环境变量
*   6\. 启动mongo
*   7\. 连接数据库
*   8\. 绑定ip说明
*   9\. 设置用户名密码
*   10\. 关闭mongo
*   11\. 使用用户名密码登录
*   12\. 配置开机自启动

  

Docker安装MongoDB

1\. 下载安装包
---------

*   这里使用免费的社区版本 ( [下载地址](https://www.mongodb.com/try/download/community))
*   版本号4.4.11，CentOS7，tgz压缩包

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库](https://s2.51cto.com/images/blog/202308/05041937_64cd5d595aa3374687.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

2\. 将安装包上传至服务器
--------------

*   这里使用xftp将安装包伤上传至虚拟机上

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_02](https://s2.51cto.com/images/blog/202308/05041937_64cd5d59726c181331.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

3\. 解压
------

1.  解压

复制

```plain
tar -zxvf mongodb-linux-x86_64-rhel70-4.4.11.tgz
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_用户名_03](https://s2.51cto.com/images/blog/202308/05041937_64cd5d59883b217859.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

2.  重命名

复制

```plain
mv mongodb-linux-x86_64-rhel70-4.4.11 mongodb
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux_04](https://s2.51cto.com/images/blog/202308/05041937_64cd5d599df3c39973.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

3.  创建数据存放文件夹、日志存放文件夹、配置文件夹

复制

```plain
mkdir -p mongodb/{data/db,log,conf}
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_配置文件_05](https://s2.51cto.com/images/blog/202308/05041937_64cd5d59aff7c69206.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_06](https://s2.51cto.com/images/blog/202308/05041937_64cd5d59c9b5927351.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

4\. 创建配置文件
----------

复制

```plain
vim mongodb/conf/mongodb.conf
```

*   添加下面的配置

复制

```plain
storage:
   dbPath: "/usr/local/soft/mongodb/data/db" #数据存放位置
systemLog:
   destination: file
   path: "/usr/local/soft/mongodb/log/mongodb.log" #日志存放位置
net:
   bindIp: 0.0.0.0 #对远程连接ip不限制
   port: 27017 #端口号
processManagement:
   fork: true #后台启动
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_07](https://s2.51cto.com/images/blog/202308/05041937_64cd5d59dea5f5679.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

5\. 修改环境变量
----------

*   将mongodb/bin目录下的可执行添加到环境变量中，方便使用mongo/bin下面的命令
*   修改profile文件

复制

```plain
vi /etc/profile
```

*   在最下面添加mongo的bin目录路径

复制

```plain
export MONGODB_HOME=/usr/local/soft/mongodb/
export PATH=$PATH:$MONGODB_HOME/bin
```

*   使文件生效

复制

```plain
source /etc/profile
```

6\. 启动mongo
-----------

*   指定配置文件启动

复制

```plain
mongod -f mongodb/conf/mongodb.conf
```

*   出现下面提示，表示启动成功，进程号是6984

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_08](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5a1320e75459.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_09](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5a37e7839462.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   查看进程

复制

```plain
ps -aux|grep mongod
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_配置文件_10](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5a5040079282.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

7\. 连接数据库
---------

*   因为没有设置用户名密码，也没有对ip做限制，本机(ip:192.168.42.111)可以直接使用`mongo`命令登录

复制

```plain
mongo
```

*   查看数据库

复制

```plain
show databases
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_11](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5a653da53554.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   客户端（ip:192.168.42.115）连接

复制

```plain
mongo mongodb://192.168.42.111:27017
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_12](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5a7c71c41695.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

8\. 绑定ip说明
----------

*   本机连接

复制

```plain
bindIp: 127.0.0.1  #只允许本机登录，其他客户都不能连接
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_用户名_13](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5a9e70540350.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_14](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5ab1d6a679.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   使用固定ip连接，这个ip可以使用一个内网的地址，保证一定程度的安全性，多个ip之间可以使用逗号分隔，比如127.0.0.1,192,168.42.111

复制

```plain
bindIp: 192.168.42.111  #只允许使用IP 192.168.42.111进行连接
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_用户名_15](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5ac7ce616275.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_16](https://s2.51cto.com/images/blog/202308/05041938_64cd5d5add15115043.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   其他客户端连接

复制

```plain
mongo mongodb://192.168.42.111:27017
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_17](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b0669379442.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   具体配置可以参考 [官方文档](https://docs.mongodb.com/manual/reference/configuration-options/#net-options)

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_18](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b213e073043.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

9\. 设置用户名密码
-----------

*   先登录

复制

```plain
mongo 192.168.42.111:27017
```

*   查看数据库

复制

```plain
show databases
```

*   切换到admin（设置密码需要切换到admin库）

复制

```plain
use admin
```

*   添加用户

复制

```plain
db.createUser( { user: "fisher", pwd: "fisher3652",roles:[{role:"root",db:"admin"}]})
```

*   查看所有用户信息

复制

```plain
show users
```

*   退出

复制

```plain
exit
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_用户名_19](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b38e7d25768.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

10\. 关闭mongo
------------

*   `一定不能使用kill -9 <pid> 进行关闭`，否则启动时会报如下错误

复制

```plain
child process failed, exited with error number 48
```

> 因为MongoDB使用mmap方式进行数据文件管理，也就是说写操作基本是在内存中进行，写操作会被每隔60秒(syncdelay设定)的flush到磁盘里。如果在这60秒内flush处于停止事情我们进行kill -9那么从上次flush之后的写入数据将会全部丢失。  
> 如果在flush操作进行时执行kill -9则会造成文件混乱，可能导致数据全丢了，启动时加了repair也无法恢复。

1.  在admin库下使用shutdownServer命令

复制

```plain
use admin
db.shutdownServer()
```

2.  使用shutdown命令

复制

```plain
mongod --shutdown
```

*   如果是指定了配置文件启动的，后面要带配置文件

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_20](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b4f3fa58845.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

3.  kill

复制

```plain
kill <mongod process ID>
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_用户名_21](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b6720b72785.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   参考 [官方文档](https://docs.mongodb.com/manual/tutorial/manage-mongodb-processes/)

11\. 使用用户名密码登录
--------------

*   使用授权模式启动mongo

复制

```plain
mongod --auth -f mongodb/conf/mongodb.conf
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_配置文件_22](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b798fe10903.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   这时候不带用户名是登录不了的

![linux登录mongodb数据库 linux连接mongodb用户名密码_配置文件_23](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5b9ceee693.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   指定host和用户名登录，端口默认27017

复制

```plain
mongo -host 192.168.42.115 -u fisher
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_linux登录mongodb数据库_24](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5bb2e3040754.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   输入密码，登录成功

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_25](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5bc84fb80920.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   这里使用授权模式登录，可以在配置文件中添加如下配置并重新启动，下次启动时可以不带`--auth`

复制

```plain
security:
    authorization: enabled
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_配置文件_26](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5bdd83630890.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

12\. 配置开机自启动
------------

*   创建mongodb.service，并添加下面配置，注意切换mongo目录为实际存放的文件路径

复制

```plain
vi /usr/lib/systemd/system/mongodb.service
```

复制

```plain
[Unit]
Description=mongodb
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
RuntimeDirectory=mongodb
PIDFile=/usr/local/soft/mongodb/data/db/mongod.lock
ExecStart=/usr/local/soft/mongodb/bin/mongod -f /usr/local/soft/mongodb/conf/mongodb.conf
ExecStop=/usr/local/soft/mongodb/bin/mongod --shutdown -f /usr/local/soft/mongodb/conf/mongodb.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_用户名_27](https://s2.51cto.com/images/blog/202308/05041939_64cd5d5bf2f7419138.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   使配置生效，重启服务器后，会发现mongo自动启动

复制

```plain
systemctl daemon-reload
systemctl start mongodb
systemctl enable mongodb
```

![linux登录mongodb数据库 linux连接mongodb用户名密码_mongodb_28](https://s2.51cto.com/images/blog/202308/05041940_64cd5d5c1234610880.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   关闭和启动服务命令

复制

```plain
service mongodb stop
service mongodb start
```

本文转自 <https://blog.51cto.com/u_16213719/6993169>，如有侵权，请联系删除。