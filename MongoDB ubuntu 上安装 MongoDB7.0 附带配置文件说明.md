开头还是介绍一下群，如果感兴趣PolarDB ,MongoDB ,MySQL ,[PostgreSQL](https://cloud.tencent.com/product/postgresql?from_column=20065&from=20065) ,Redis ，Oracle ,Oceanbase 等有问题，有需求都可以加群群内有各大[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)行业大咖，CTO，可以解决你的问题。加群请加微信号 liuaustin3 （共1250人左右 1 + 2 + 3 +4）新人会进入3群 （3群准备关闭自由申请）+ 4群

每日感悟

代码语言：javascript

复制

```javascript
存在就是合理的，这句话是剥夺你理解内部原理的障眼法，很多人把 了解 和 理解 混淆了，如同知道了无数生活“真理”，而依然过不好这一生。

```

最近除了中国好声音停播这个好消息外，我最近关注MongoDB 7.0,所以安装看看有什么可以玩的新东西，基于CENTOS 的不能再用，全面转向了ubuntu 22.04，所以这次有两个部分的工作，

1 逐步熟悉ubuntu 22.04

2 看看MongoDB 7.0 的配置文件的变化

3 搭建复制集有什么区别

最后把相关开源的mongodb的配置文件贴上附带解释。最近的一些新的知识，没有跟上，所以对于新版本的MongoDB 的重新上路，后面会把继续学习的部分写成一个系列。

首先的不同点，如果是从MongoDB 4.X ,直接来接触MongoDB 7.0的话，第一个感受是，安装的文件变化了，之前在安装中，安装包包含了MongoDB的执行文件，客户端的文件，还有基础的工具文件，所以下载一个二进制包就可以了，但是在7.0 不可以，你至少需要下载三个部分

1 MongoDB 二进制文件包

2 MongoDB shell 客户端

3 MongoDB Tools 工具包

所以如果要完成原有4.X 的工作，现在需要下载的软件包为3个。

配置文件方面的变化

1 在systemLog 部分并未有较大的变化，需要注意从 MongoDB 4.4后，timeStampFormat 部分不再支持 ctime , 配置时需要注意默认的值改为ISO8601-LOCAL

2 在storage 配置部分，关于journal 日志的部分的配置变更移除了storage.journal.enabled 的部分，也移除了 storage.indexBuildRetry的部分，关于 maxCacheOverflowFileSizeGB，这个选项也从4.4后失效了。同时关于 storage.syncPeriodSecs 部分不要进行人工设置。

同时从MonogDB 4.4 添加了storage.oplogMinRetntionHours，通过此选项来设置oplog 保留的时间，这里默认值为0 。

在storage.wiredTiger.engineConfig.JournalCompressor 从4.2 添加了zstd的压缩模式支持，storage.wiredTiger.engineConfig.zstdCompressionlevel 可以设置压缩的比率从1 到22 默认压缩的等级为 6 ， 这个配置选项从MongoDB 5.0开始。

3 replication 项目中添加了 replication.enableMajorityReadConcern 选项从5.0开始不能在进行变动，默认值为 true，这里需要注意，在这选项中，如果你的3节点PSA 模式中有arbiter 节点的出现，那么可能有导致性能的问题，所以建议在5.0 版本后，不建议使用arbiter 代替3节点 replica.

4 Security.javascriptEnabled 通过从4.4 版本开始JAVASCRIPT 可以在系统中运行或不运行可以进行设置。security.clusterIpSourceAllowlist 是从Mongo 5.0 提供的参数，通过参数可以设置可以安全的确认复制集，或分片中主机的地址是否是安全，防止欺骗性的加入集群的可能性。

代码语言：javascript

复制

```javascript
#This is mongodb config file for replica set
#systemlog file

systemLog:
  traceAllExceptions: false
  quiet: false
  logRotate: rename
  destination: file
  logAppend: true
  path: /data/mongo1/mongod.log
  timeStampFormat: iso8601-local
  component:
     index:
       verbosity: 1

#storage
storage:
  dbPath: /data/mongo1/
  directoryPerDB: true
  wiredTiger:
     engineConfig:
        cacheSizeGB: 2
        journalCompressor: zstd
        directoryForIndexes: true
        zstdCompressionLevel: 10


processManagement:
  fork: true
  timeZoneInfo: /usr/share/zoneinfo
  pidFilePath: /data/mongo1/mongod.pid

# network interfaces
net:
  port: 27017
  bindIp: 192.168.198.100, 127.0.0.1
  bindIpAll: false
  maxIncomingConnections: 200
  wireObjectCheck: true
  unixDomainSocket:
     enabled: true
     pathPrefix: /tmp
     filePermissions: 0700

#security options
security:
   keyFile: /data/keyfile
   authorization: enabled
   javascriptEnabled: true
   clusterIpSourceAllowlist:
      - 192.168.198.0/24
      - 127.0.0.1
#  enableEncrypthion only enterprise version have it 

replication:
   oplogSizeMB: 10240
   replSetName: mongo7
   enableMajorityReadConcern: true




```

除此以外在mongo4.4后关于慢查询的部分添加了operationProfiling.filter 可以通过这个部分来过滤慢查询语句，例如filter:''{op:"query",millis: {$gt:500}} ，通过过滤可以自定义一些想找到的语句来进行问题的解决。

大家的过程比较简单

1 3个节点分别提供好相关的配置文件

2 启动第一个节点，并键入用户名密码，需要管理员的权限

3 启动其他的节点，并通过命令来逐一添加其他节点。

代码语言：javascript

复制

```javascript
openssl rand -base64 768 > keyfile
chmod 400 keyfile
sudo dpkg -i mongodb-mongosh_1.10.5_amd64.deb 
sudo dpkg -i mongodb-database-tools-ubuntu2204-x86_64-100.8.0.deb 
```

在第一次搭建的情况下，请先去掉复制方面的配置，否则无法添加用户，在添加用户后，直接执行下方的命令，将3个节点标定为复制集，然后直接初始化，即可。

![](https://developer.qcloudimg.com/http-save/yehe-5669671/a7449e0d086c87e42aec50010bc5b8cc.png)

![](https://developer.qcloudimg.com/http-save/yehe-5669671/1d772a7289a33e832d6f1ef8436f4175.png)

![](https://developer.qcloudimg.com/http-save/yehe-5669671/18a4b042ad04c13c66e6d51e026b5a78.png)

然后启动 1 2 3 mongodb，登陆到我们刚才加入账号的节点

mongosh mongodb://192.168.198.100:27017/admin -u root -p 1234.Com

代码语言：javascript

复制

```javascript
 config_rs= { _id:"mongo7", members:[
{_id:0,host:"192.168.198.100:27017",priority:100},
{_id:1,host:"192.168.198.100:27027",priority:80},
{_id:2,host:"192.168.198.100:27037",priority:60}]}

rs.initiate(config_rs)
```

![](https://developer.qcloudimg.com/http-save/yehe-5669671/fa95de898f0b56c5470dd6a31a6f5d17.png)

![](https://developer.qcloudimg.com/http-save/yehe-5669671/5c0dbdd4640f4c02a9b93070148d0b35.png)

随着7.0 复制集搭建完成，后续的关于7.0的研究会慢慢展开

本文转自 <https://cloud.tencent.com/developer/article/2330109>，如有侵权，请联系删除。