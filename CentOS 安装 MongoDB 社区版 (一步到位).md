## CentOS 安装 MongoDB 社区版 (一步到位)

**mongodb官方地址**

```sh
https://www.mongodb.com/try/download/community
```

选择下载版本

![img](https://img-blog.csdnimg.cn/bdca5a97afbe4a5ea004defb674214b6.png)

![img](https://img-blog.csdnimg.cn/dee8443c8eed4dc49a8fd76683f08c76.png)

版本选择好后 点击  CopyLink 复制 连接

服务器执行 下载命令:

```sh
# 进入文件夹
cd /usr/local
 
# 下载 MongoDB 社区版本
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-7.0.14.tgz
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-7.0.2.tgz

#下载mongodb备份工具
wget  https://fastdl.mongodb.org/tools/db/mongodb-database-tools-rhel70-x86_64-100.10.0.tgz
或ubuntu下载
wget  https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu2204-x86_64-100.10.0.tgz
 
# 解压
tar -zxvf mongodb-linux-x86_64-rhel70-7.0.2.tgz
 
# 重命名文件夹
mv mongodb-linux-x86_64-rhel70-7.0.2 mongodb
 
# 进入目录
cd mongodb/
 
# 创建三个文件夹
mkdir data data/db data/log
 
# 设置可读写权限
sudo chmod 666 data/db data/log/

#设置mongodb用户和用户组
sudo useradd -r mongodb
sudo groupadd mongodb

chown -R mongodb:mongodb /opt/mongodb/log
chown -R mongodb:mongodb /opt/mongodb/data
```

在 mongodb 目录下 

执行 vim mongodb.conf 

并且添加以下内容:

```sh
systemLog:
  destination: file
  path: /mongodb/log/mongodb.log # log path
  logAppend: true
storage:
  dbPath: /mongodb/data # data directory
  engine: wiredTiger  #存储引擎，默认值就是wiredTiger
  journal:            #journal日志配置
    commitIntervalMs: 100 #mongod进程在日志操作之间允许的最大时间（以毫秒为单位）。值可以从1到500毫秒不等。较低的值会增加日志的耐用性，而牺牲了磁盘性能。在WiredTiger上，默认的日志提交间隔是100毫秒。此外，包含或暗示j:true的写入将导致期刊立即同步。
net:
  bindIp: 0.0.0.0
  port: 27017
processManagement:
  fork: true
```

配置环境变量

```sh
# 配置文件编辑
sudo vim /etc/profile
 
# 在 unset i 上面 添加以下内容
export MONGODB_HOME=/usr/local/soft/mongodb
PATH=$PATH:$MONGODB_HOME/bin
 
# 保存退出 
:wq
 
# 刷新 配置 文件 
source /etc/profile
 
# 启动 MongoDB
mongod -f mongodb.conf

```

启动 成功

![img](https://img-blog.csdnimg.cn/ebddbae9945b4e5b9a3afe4764894ba3.png)

下载 MongoDB 连接 工具

**mongodb连接工具**

```sh
https://www.mongodb.com/try/download/compass
```

点击 CopyLink 复制下载链接

![img](https://img-blog.csdnimg.cn/3fcc667555cb4d6fb7dde635e8901edc.png)





执行

```sh
# 下载 工具
wget https://downloads.mongodb.com/compass/mongosh-2.0.2-linux-x64.tgz
wget https://downloads.mongodb.com/compass/mongosh-2.3.1-linux-x64.tgz 
# 解压
tar -zxvf mongosh-2.0.2-linux-x64.tgz
 
# 重命名文件夹
mv mongosh-2.0.2-linux-x64 mongosh
 
# 进入文件夹
cd mongosh/bin/

```

输入 ./mongosh  进入命令行模式

输入 use admin

输入 db.createUser({ user: "root", pwd: "BMJhMKpzjyn12", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })

![img](https://img-blog.csdnimg.cn/87d0f2927a4a4ea8856e0bbd77bb725f.png)

输入:db.grantRolesToUser('root',[{ role: "root", db: "admin" }])

![img](https://img-blog.csdnimg.cn/159f29eed36e4fd1afdf28c20127b71b.png)

授权成功        exit 退出 数据库命令行工具

添加 服务器 自启动 :

```sh
# 关闭 MongoDB 服务
mongod -f /usr/local/mongodb/mongodb.conf  --shutdown
```

![img](https://img-blog.csdnimg.cn/0a2b73f4fb9c4bc797678f2a69929cf1.png)

关闭 成功  

添加 自启动:

```sh
# 编辑 文件
sudo vim /lib/systemd/system/mongodb.service
 
# 输入以下内容
 
[Unit]
    Description=mongodb
    After=network.target remote-fs.target nss-lookup.target
[Service]
    Type=forking
    ExecStart=/usr/local/mongodb/bin/mongod -f /usr/local/mongodb/mongodb.conf
    ExecReload=/bin/kill -s HUP $MAINPID
    ExecStop=/usr/local/mongodb/bin/mongod -f /usr/local/mongodb/mongodb.conf --shutdown
    PrivateTmp=true
[Install]
    WantedBy=multi-user.target
```

然后 执行 以下 命令

```sh
# 启动 mongodb
[root@VM-12-12-centos mongodb]# systemctl start mongodb.service
 
# 查看 服务 状态
[root@VM-12-12-centos mongodb]# systemctl status mongodb.service
● mongodb.service - mongodb
   Loaded: loaded (/usr/lib/systemd/system/mongodb.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-11-06 20:32:54 CST; 9s ago
  Process: 13264 ExecStart=/usr/local/mongodb/bin/mongod -f /usr/local/mongodb/mongodb.conf (code=exited, status=0/SUCCESS)
 Main PID: 13266 (mongod)
   CGroup: /system.slice/mongodb.service
           └─13266 /usr/local/mongodb/bin/mongod -f /usr/local/mongodb/mongodb.conf
 
Nov 06 20:32:53 VM-12-12-centos systemd[1]: Starting mongodb...
Nov 06 20:32:53 VM-12-12-centos mongod[13264]: about to fork child process, waiting until server is ready for connections.
Nov 06 20:32:53 VM-12-12-centos mongod[13264]: forked process: 13266
Nov 06 20:32:54 VM-12-12-centos mongod[13264]: child process started successfully, parent exiting
Nov 06 20:32:54 VM-12-12-centos systemd[1]: Started mongodb.
 
# 添加开机自启动
[root@VM-12-12-centos mongodb]# systemctl enable mongodb.service
Created symlink from /etc/systemd/system/multi-user.target.wants/mongodb.service to /usr/lib/systemd/system/mongodb.service.
 
# 刷新一次配置
[root@VM-12-12-centos mongodb]# systemctl daemon-reload

```

常规 服务  命令:

```sh
# 服务启动
systemctl start mongodb.service
 
# 服务重启
systemctl restart mongodb.service
 
# 服务停止
systemctl stop mongodb.service
```



monogdb常用命令

```sh
# 查询数据总数
db.getCollection("库名").count()
# ex: db.getCollection("order").count()

# 删除指定时间之前的数据（createTime为字符串时间）
db.库名.remove({"createTime":{$lte:"2021-09-28 00:00:00"}})
# ex: db.order.remove({"createTime":{$lte:"2021-09-28 00:00:00"}})
# 响应结果： WriteResult({ "nRemoved" : 130834, "writeConcernError" : [ ] })
```

+ 导出数据

```sh
# 导出数据    mongoexport --help
# -h 数据库地址，MongoDB 服务器所在的 IP 与 端口，如 127.0.0.1:27017
# -d 指明使用的数据库实例，如 test-db
# -c 指明要导出的集合，如 order_20211104
# -o 指明要导出的文件名，如 ./order.json  注意是文件而不是目录，目录不存在时会一同新建
mongoexport -h 127.0.0.1:27017 -d test-db -c order_20211104 -u admin -p=123456 -o ./order.json --authenticationDatabase admin

# 导入数据
mongoimport -h 127.0.0.1:27017 -d test-db -c order_20211104 -u admin -p=123456 ./order.json --authenticationDatabase admin
```



centos启动mongodb报错解决

>mongod: error while loading shared libraries: libcrypto.so.10: cannot open shared object file: No such file or directory

```sh
https://blog.csdn.net/qq_46001933/article/details/129685983
问题原因是 是 centos 8 跟centos7 不太一样了

缺少这个库文件

解决方案：
下载库：
wget https://gitee.com/wang-huamao/soft/raw/35d99d8294ba79e8d2f839e4bd93c15223115aab/linux/compat-openssl10-1.0.2o-4.el8_6.x86_64.rpm

安装：
rpm -ivh compat-openssl10-1.0.2o-4.el8.x86_64.rpm
dnf install compat-openssl10

然后再启动mongodb
./mongod -f mongodb.conf
```



### ubuntu安装monogdb

+ 步骤：

> 1、导入 MongoDB 公钥

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -

```

> 2、创建 `/etc/apt/sources.list.d/mongodb-org-7.0.list` 文件，加入 MongoDB 存储库

```sh
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

```

> 更新包列表并安装 MongoDB：

```sh
sudo apt update
sudo apt install -y mongodb-org

```

> 安装mongodb报错：mongodb-org-mongos : 依赖: libssl1.1 (>= 1.1.1) 但无法安装它

```sh
#在 Ubuntu 22.04 中，libssl1.1 已不再默认包含在系统中，原因是 Ubuntu 22.04 默认使用 OpenSSL 3.0，但 mongodb-org-mongos 依赖 libssl1.1。因此，我们需要手动下载并安装 libssl1.1 以解决这个依赖问题。
#以下是解决方法的步骤：
1、手动下载 libssl1.1 包
libssl1.1 可以从 Ubuntu 20.04 (Focal) 软件仓库中下载。执行以下步骤来手动安装它：
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb
2、安装 libssl1.1
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
3、修复依赖关系
sudo apt --fix-broken install
```

> 再次安装 MongoDB
>
> 在成功安装 `libssl1.1` 后，你可以继续安装 MongoDB：

```sh
sudo apt update
sudo apt install -y mongodb-org

```

> 启动 MongoDB 服务
>
> #安装完成后，启动 MongoDB 服务

```sh
sudo systemctl start mongod
sudo systemctl enable mongod
```

![image-20240930105159928](C:\Users\EDY\AppData\Roaming\Typora\typora-user-images\image-20240930105159928.png)

### MongoDB7.0的安装

#### MongoDB 7.0新特性

- 2024年8月15日，MongoDB正式发布7.0版本，截止目前最新版本为7.0.6。
- [MongoDB7.0特性](https://www.mongodb.com/docs/v7.0/release-notes/7.0/)
- [阿里云关于MongoDB7.0的特性说明](https://help.aliyun.com/zh/mongodb/product-overview/features-of-mongodb-7-0?spm=a2c4g.11186623.0.0.2acc36f3yP2Mr9)，该文档中也包含MongoDB其它版本的特性说明
- [MongoDB Software Lifecycle Schedules](https://www.mongodb.com/legal/support-policy/lifecycles)

#### 安装MongoDB

- mongodb的安装方法可以查看[官方文档](https://www.mongodb.com/docs/v7.0/administration/install-community/)
- [MongoDB下载地址](https://www.mongodb.com/try/download/community)，选择合适的版本、平台和包类型

```sh
#下载MongoDB
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel80-7.0.6.tgz
# 解压
tar -zxvf mongodb-linux-x86_64-rhel80-7.0.6.tgz
# 方便起见，创建软连接
ln -s mongodb-linux-x86_64-rhel80-7.0.6 mongodb

# 修改/etc/profile，添加环境变量，方便执行MongoDB命令
export MONGODB_HOME=/usr/local/soft/mongodb
PATH=$PATH:$MONGODB_HOME/bin

#重新加载环境变量
source /etc/profile

#查看版本，检查命令是否可用
mongod --version

#创建dbpath和logpath的存储目录
mkdir -p /mongodb/data /mongodb/log

```

#### 安装mongoDB时可能遇到的问题

- 1. 启动mongodb服务时，提示`mongod: error while loading shared libraries: libcrypto.so.1.1: cannot open shared object file: No such file or directory`

> 解决方法：

```sh
wget  https://www.openssl.org/source/openssl-1.1.1w.tar.gz
tar -zxvf openssl-1.1.1w.tar.gz
cd openssl-1.1.1w
./config
# 如果make时提示 /bin/sh: gcc: command not found，需要先安装gcc：sudo yum install gcc -y
make && make install
ln -s /usr/local/lib64/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
ln -s /usr/local/lib64/libssl.so.1.1 /usr/lib64/libssl.so.1.1

```

#### 启动mongodb服务

- 命令参数启动

```sh
mongod --port=27017 --dbpath=/mongodb/data --logpath=/mongodb/log/mongodb.log --bind_ip=0.0.0.0 --fork
    # 参数说明
    --port: 指定端口，默认为27017
    --dbpath: 指定数据文件存放目录
    --logpath: 指定日志文件，注意是指定文件不是目录
    --bind_ip: 默认只监听localhost网卡
    --fork: 后台启动

```

- 也可以将上面的参数写到配置文件中，如`/mongodb/conf/mongo.conf`文件，必须是yaml格式

```sh
systemLog:
  destination: file
  path: /mongodb/log/mongodb.log # log path
  logAppend: true
storage:
  dbPath: /mongodb/data # data directory
  engine: wiredTiger  #存储引擎，默认值就是wiredTiger
  journal:            #journal日志配置
    commitIntervalMs: 100 #mongod进程在日志操作之间允许的最大时间（以毫秒为单位）。值可以从1到500毫秒不等。较低的值会增加日志的耐用性，而牺牲了磁盘性能。在WiredTiger上，默认的日志提交间隔是100毫秒。此外，包含或暗示j:true的写入将导致期刊立即同步。
net:
  bindIp: 0.0.0.0
  port: 27017
processManagement:
  fork: true

```

- 将命令行参数直接转换为yaml:`--outputConfig`

```sh
$ mongod --port=27017 --dbpath=/mongodb/data --logpath=/mongodb/log/mongodb.log --bind_ip=0.0.0.0 --fork --outputConfig

```

```sh
net:
  bindIp: 0.0.0.0
  port: 27017
outputConfig: true
processManagement:
  fork: true
storage:
  dbPath: /mongodb/data
systemLog:
  destination: file
  path: /mongodb/log/mongodb.log

```

> 删除`outputConfig: true`这一行，然后将其余内容复制到mongo.conf中

- 关于配置参数的详细信息可以查看[官方文档](https://www.mongodb.com/docs/v7.0/reference/configuration-options/)

```sh
# 启动mongo服务
mongod -f /mongodb/conf/mongo.conf

# 关闭mongo服务，注意：macos下不支持 --shutdown
mongod -f /mongodb/conf/mongo.conf --shutdown

```

#### shell客户端mongosh

- 从mongodb6开始不再支持mongo命令，而是需要使用mongosh命令，关于mongosh命令的使用可以查看[官方文档](https://www.mongodb.com/docs/mongodb-shell/)
- mongosh命令的使用方式与mongo命令基本一致
- 下载地址：[mongosh下载地址](https://www.mongodb.com/try/download/shell)

```sh
# 下载安装包
# openssl11是centos7的版本，目前支持centos8，但更推荐使用centos8时下载openssl3的版本
wget https://downloads.mongodb.com/compass/mongosh-2.2.2-linux-x64-openssl11.tgz
# 解压
tar -zxvf mongosh-2.2.2-linux-x64-openssl11.tgz
# 创建软连接
ln -s mongosh-2.2.2-linux-x64-openssl11 mongosh

# 修改/etc/profile，添加环境变量，方便执行MongoShell命令
export MONGODB_SHELL_HOME=/usr/local/soft/mongosh
PATH=$PATH:$MONGODB_SHELL_HOME/bin

#重新加载环境变量
source /etc/profile

# 查看版本
mongosh --version

# 连接mongodb server端
mongosh --host=127.0.0.1 --port=27017
# --host: mongodb server端ip地址
# --port: mongodb server端口

```

- mongosh常用命令

| 命令                            | 说明                             |
| ------------------------------- | -------------------------------- |
| show dbs 或 show databases      | 显示数据库                       |
| use 数据库名                    | 切换数据库，如果不存在创建数据库 |
| db.dropDatabase()               | 删除数据库                       |
| show collections 或 show tables | 显示当前数据库的集合列表         |
| db.集合名.stats()               | 查看集合详情                     |
| db.集合名.drop()                | 删除集合                         |
| show users                      | 显示当前数据库的用户列表         |
| show roles                      | 显示当前数据库的角色列表         |
| show profile                    | 显示最近发生的操作               |

| load(“xxx.js”)      | 执行一个JavaScript脚本文件 |
| ------------------- | -------------------------- |
| exit 或 quit        | 退出                       |
| help                | 查看mongodb支持哪些命令    |
| db.help()           | 查询当前数据库支持的方法   |
| db.集合名.help()    | 显示集合的帮助信息         |
| db.version()        | 查看数据库版本             |
| cls                 | 清屏                       |
| db.shutdownServer() | 关闭mongodb server端       |
|                     |                            |
|                     |                            |

#### 安全认证

- 创建管理员

```sh
# 设置管理员用户名密码需要切换到admin库
use admin
#显示可设置权限
show roles

#创建管理员，授予root角色
db.createUser({user:"root",pwd:"password",roles:["root"]})

#查看当前数据库所有用户信息
show users
#显示所有用户
db.system.users.find()

```

- 常用角色

| 权限名               | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| read                 | 允许用户读取指定数据库                                       |
| readWrite            | 允许用户读写指定数据库                                       |
| dbAdmin              | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile |
| dbOwner              | 允许用户在指定数据库中执行任意操作，增、删、改、查等         |
| userAdmin            | 允许用户向system.users集合写入，可以在指定数据库里创建、删除和管理用户 |
| clusterAdmin         | 只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限 |
| readAnyDatabase      | 只在admin数据库中可用，赋予用户所有数据库的读权限            |
| readWriteAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的读写权限          |
| userAdminAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限     |
| dbAdminAnyDatabase   | 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限       |
| root                 | 只在admin数据库中可用。超级账号，超级权限                    |

- 创建数据库用户

```sh
admin> use mydb
switched to db mydb

# 默认认证数据库为当前数据库，下面的授权等同于 { role: 'dbOwner', db: 'mydb' }
mydb> db.createUser({user:"mytest",pwd:"123456",roles:["dbOwner"]})
{ ok: 1 }

# 显示当前数据库下的用户
mydb> show users
[
  {
    _id: 'mydb.mytest',
    userId: UUID('8bc42a74-5d84-4849-af25-09fdcbdfd03a'),
    user: 'mytest',
    db: 'mydb',
    roles: [ { role: 'dbOwner', db: 'mydb' } ],
    mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
  }
]

```

- 重置用户密码

```sh
mydb> db.changeUserPassword("mytest", "password")
{ ok: 1 }
# 或者
mydb> db.updateUser("mytest", {pwd: "password"})
{ ok: 1 }
```

- 为用户添加角色

```sh
# 假设已经创建了用户mytest，需要重新赋予其角色
mydb> db.grantRolesToUser( "mytest" , [
 { role: "dbAdmin", db: "mydb" } ,
 { role: "userAdmin", db: "mydb"}
 ])
```

- 删除用户的指定角色

```sh
mydb> db.revokeRolesFromUser( "mytest" , [
  { role: "dbAdmin", db: "mydb" }
  ])
```

- 删除用户

```sh
mydb> db.dropUser("mytest")
{ ok: 1 }
mydb> show users
[]

# 删除全部用户
mydb> db.dropAllUsers()
```

#### MongoDB启用鉴权

- 默认情况下，MongoDB不会启用鉴权，以鉴权模式启动MongoDB有两种方法

> 命令行参数增加 `--auth`

```sh
mongod -f /mongodb/conf/mongo.conf --auth
```

> 配置文件中加上如下内容

```sh
security:
  authorization: enabled
```

- 启用鉴权之后，连接MongoDB的相关操作都需要提供身份认证

```sh
mongosh --host=127.0.0.1 --port=27017 -u root -p password --authenticationDatabase=admin
# -u: 用户名
# -p: 密码
# --authenticationDatabase: 指定认证数据库
```

#### mongosh连接mongodb server端的方式

- 参数方式

```sh
# 可以通过 mongosh --help 查看帮助
mongosh --host=127.0.0.1 --port=27017 -u root -p password --authenticationDatabase=admin
# 指定连接的数据库，这里指定连接到mydb数据库，如果不指定，默认连接到test数据库
mongosh mydb --host=127.0.0.1 --port=27017 -u root -p password --authenticationDatabase=admin
```

- 混合方式

```sh
# ip+端口方式连接
mongosh 127.0.0.1:27017
# ip+端口方式连接，后面可以加上各种参数配置
mongosh 127.0.0.1:27017 -u root -p password --authenticationDatabase=admin
# ip+端口方式连接，同时可以指定连接的数据库，这里指定连接到mydb数据库，如果不指定，默认连接到test数据库
mongosh 127.0.0.1:27017/mydb
```

- uri方式

```sh
mongosh mongodb://127.0.0.1:27017
# 指定连接数据库
mongosh mongodb://127.0.0.1:27017/mydb
# 带认证方式连接
mongosh "mongodb://root:password@127.0.0.1:27017/mydb?authSource=admin"
# 带认证方式连接，同时指定readPreference为primaryPreferred，即读取数据时优先从主节点读取数据
mongosh "mongodb://root:password@127.0.0.1:27019/mydb?authSource=admin&readPreference=primaryPreferred"
```

- 如果只是连接本机的server端，而且端口为27017，可以省略host和port

```sh
mongosh
```

### 解决连接MongoDB时出现的 vm.max_map_count is too low 的问题

- 默认的`vm.max_map_count`值为`65530`，如果需要开启MongoDB的分片功能，需要将`vm.max_map_count`设置为较高的值，通常推荐为`1048576`
- 查看当前`vm.max_map_count`的值

```sh
sysctl vm.max_map_count
```

- 临时增加`vm.max_map_count`的值

```sh
sysctl -w vm.max_map_count=1048576
```

- 永久增加`vm.max_map_count`的值

```sh
# 修改/etc/sysctl.conf文件
echo "vm.max_map_count = 1048576" >> /etc/sysctl.conf
# 使配置生效
sysctl -p
```

- 重启MongoDB才会生效

#### mongodb7.0.14优化配置

```yaml
storage:
  dbPath: /opt/mongodb/data/db  # 数据库文件存放路径
  wiredTiger:
    engineConfig:
      cacheSizeGB: 2  # 分配给 WiredTiger 引擎的缓存，建议为系统内存的 50%-70%
      directoryForIndexes: false  # 如果没有单独的磁盘用于索引存储，设置为false
      journalCompressor: snappy  # 日志压缩算法，使用snappy提供高效的压缩

    collectionConfig:
      blockCompressor: snappy  # 集合块压缩，减少磁盘占用，推荐snappy

    indexConfig:
      prefixCompression: true  # 索引前缀压缩，提高索引效率

processManagement:
  fork: true  # 启动为守护进程
  pidFilePath: /opt/mongodb/mongod.pid  # PID 文件存放路径

net:
  port: 27017  # MongoDB服务监听端口
  bindIp: 127.0.0.1  # 仅允许本地访问，提升安全性
  maxIncomingConnections: 500  # 最大连接数，根据应用场景调整


systemLog:
  destination: file
  path: /opt/mongodb/data/log/mongod.log  # 日志文件路径
  logAppend: true  # 日志追加模式
  verbosity: 1  # 设置为1，减少不必要的日志信息，适合生产环境

operationProfiling:
  mode: slowOp  # 记录慢查询

# 连接池和并发控制
setParameter:
  wiredTigerConcurrentReadTransactions: 64  # 并发读事务数
  wiredTigerConcurrentWriteTransactions: 64  # 并发写事务数
```

> **`wiredTiger.engineConfig.cacheSizeGB`**：设置WiredTiger缓存大小，建议设置为系统可用内存的50%-70%。对于单机环境，通常会占用8GB的内存（如果你的系统内存较多，可以增加）。缓存越大，MongoDB能够在内存中处理的数据越多，减少磁盘 I/O。
>
> **`journalCompressor` 和 `blockCompressor`**：使用 `snappy` 压缩日志和数据块，`snappy` 提供较好的压缩性能，同时占用较少的CPU资源。通过数据压缩减少磁盘存储空间并提升 I/O 性能。
>
> **`net.bindIp`**：设置为 `127.0.0.1`，限制MongoDB仅接受来自本地的连接，增强单机环境的安全性。对于生产环境，请根据需要设置合适的IP范围。
>
> **`operationProfiling.slowOpThresholdMs`**：将慢查询阈值设为100ms。任何超过100ms的查询将记录下来以供分析。这有助于识别并优化低效查询。
>
> **`wiredTigerConcurrentReadTransactions` 和 `wiredTigerConcurrentWriteTransactions`**：这些参数控制WiredTiger引擎的并发事务处理数量。根据单机的CPU和内存资源，这里将并发读写限制为64个线程，适合普通的单机环境。如果有更多资源，可以适当增加这个值。
>
> **`maxConnsPerHost`**：限制最大连接数为500。单机部署中，这个值可以有效限制内存消耗并防止过多的连接影响性能。
>
> **`security.authorization`**：启用用户认证，确保MongoDB服务不被未授权的访问，特别是在开发或生产环境中。
>
> **`systemLog.verbosity`**：设置日志详细级别为1，减少不必要的日志输出，避免过多的日志记录影响性能。
>
> **`storage.journal.enabled`**：启用WiredTiger的日志机制以保证数据的可靠性。如果应用场景对数据安全要求较高，这是必不可少的。
>
> **`cloud.monitoring.free.state`**：禁用MongoDB的云监控服务，减少外部连接，优化本地性能。

#### 安装了mongodb对应bin目录下没有mongodump和mongorestore命令

> 今天在学习MongoDB数据库过程中，到了数据备份和恢复时，发现原来下载的数据库文件缺失很多工具，需要单独下载，对应补充工具的下载链接：https://www.mongodb.com/try/download/database-tools

![img](https://i-blog.csdnimg.cn/blog_migrate/131685ac2909e8baae22891ec99aeeb3.png)

























































































































































































































































































































































