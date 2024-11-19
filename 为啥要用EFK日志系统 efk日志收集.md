我整理的一些关于【IT人转技术管理】的项目学习资料（附讲解～～）和大家一起分享、学习一下：

 [https://d.51cto.com/eDOcp1](https://d.51cto.com/eDOcp1)

  

👔01 EFK日志收集系统概述
----------------

⚽1.ELK诞生的背景
-----------

### 💡1.1 没有ELK分析日志前

> **没有日志分析工具之前，运维工作存在哪些痛点？  
> 痛点1、生产出现故障后，运维需要不停的查看各种不同的日志进行分析？是不是毫无头绪？  
> 痛点2、项目上线出现错误，如何快速定位问题？如果后端节点过多、日志分散怎么办？  
> 痛点3、开发人员需要实时查看日志但又不想给服务器的登陆权限，怎么办？难道每天帮开发取日志？  
> 痛点4、如何在海量的日志中快速的提取我们想要的数据？比如：PV、UV、TOP10的URL？如果分析的日志数据量大，那么势必会导致查询速度慢、难度增大，最终则会导致我们无法快速的获取到想要的指标。  
> 痛点5、CDN公司需要不停的分析日志，那分析什么？主要分析命中率，为什么？因为我们给用户承诺的命中率是90%以上。如果没有达到90%，我们就要去分析数据为什么没有被命中、为什么没有被缓存下来。**\*

### 💡1.2 使用ELK分析日志后

> **如上所有的痛点都可以使用日志分析系统ELK解决，通过ELK，将运维所有的服务器日志，业务系统日志都收集到一个平台下，然后提取想要的内容，比如错误信息，警告信息等，当过滤到这种信息，就马上告警，告警后，运维人员就能马上定位是哪台机器、哪个业务系统出现了问题，出现了什么问题。**

⚽2.ELK技术栈是什么
------------

### 💡2.1 什么是ELK

> **其实 ELK 不是一个单独的技术，而是一套技术的组合，是由 elasticsearch、logstash、kibana 组合而成的。 ELK 是一套开源免费、功能强大的日志分析管理系统。ELK**  
> **可以将我们的系统日志、网站日志、应用系统日志等各种日志进行收集、过滤、清洗，然后进行集中存放并可用于实时检索、分析。**

*   E: elasticsearch 数据存储；
*   L: logstash 数据采集、数据清洗、数据过滤；
*   K: kibana 数据分析、数据展示；

### 💡2.2 什么是EFK

> **简单来说就是将 Logstash 替换成了 filebeat，那为什么要进行替换？ 因为 logstash 是基于 JAVA  
> 开发的，在收集日志时会大量的占用业务系统资源，从而影响正常线上业务。 而替换成 filebeat  
> 这种较为轻量的日志收集组件，会让业务系统的运行更加的稳定。**

![为啥要用EFK日志系统 efk日志收集_elasticsearch](https://s2.51cto.com/images/blog/202404/08055249_661315b17f62e86449.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡2.3 什么是ELFK（+kafka）

![为啥要用EFK日志系统 efk日志收集_数据_02](https://s2.51cto.com/images/blog/202404/08055249_661315b19625c17948.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡2.4 EFK收集哪些日志

*   代理： Haproxy、Nginx
*   web：Nginx、Tomcat、Httpd、PHP
*   db：mysql、redis、mongo、elasticsearch
*   存储：nfs、glusterfs、fastdfs
*   系统：message、security
*   业务：app

👔02 Elasticsearch入门
--------------------

⚽1.ES基本介绍
---------

### 💡1.1 ES是什么

> **Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎。**

### 💡1.2 ES主要功能

> **数据存储、数据搜索、数据分析。**

### 💡1.3 ES相关术语

#### 1.3.1 文档 Document

**Document 文档就是用户存在 es 中的一些数据，它是 es 中存储的最小单元。（类似于表中的一行数据。）注意：每个文档都有一个唯一的 ID 表示，可以自行指定，如果不指定 es 会自动生成。**

![为啥要用EFK日志系统 efk日志收集_java_03](https://s2.51cto.com/images/blog/202404/08055249_661315b1c40ed8046.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 1.3.2 索引 Index

**索引其实是一堆文档 Document 的集合。（它类似数据库的中的一个表）**

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_04](https://s2.51cto.com/images/blog/202404/08055249_661315b1e899e40492.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 1.3.3 字段 Filed

**在 ES 中，Document就是一个 Json Object，一个Json Object其实是由多个字段组成的，每个字段它有不同的数据类型。**

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_05](https://s2.51cto.com/images/blog/202404/08055250_661315b20736a65262.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 1.3.4 字段的数据类型

*   字符串：text、keyword。
*   数值型：long,integer,short,byte,double,float
*   布尔：boolean
*   日期：date
*   二进制：binary
*   范围类型：integer\_range,float\_range,long\_range,double\_range,date\_range

### 💡1.4 ES术语总结

**ES索引、文档、字段关系小结：  
一个索引里面存储了很多的 Document 文档，一个文档就是一个json object，一个json object是由多个不同或相同的 filed 字段组成；**

![为啥要用EFK日志系统 efk日志收集_elasticsearch_06](https://s2.51cto.com/images/blog/202404/08055250_661315b226d5730979.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

⚽2.ES单机安装部署
-----------

### 💡2.1 安装方式两种

*   二进制安装：解压启动
*   rpm安装包：安装配置启动

### 💡2.2 rpm方式安装

#### 2.2.1 首先将下载好的rpm安装包上传到linux系统上

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_07](https://s2.51cto.com/images/blog/202404/08055250_661315b269e9963759.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 2.2.2 安装java环境，再本地安装elasticsearch

登录后复制

```plain
yum -y install java
yum  -y localinstall  elasticsearch-7.4.0-x86_64.rpm
```

#### 2.2.3 elasticsearch的配置文件

登录后复制

```plain
[root@es-node1 ~]# vim /etc/elasticsearch/elasticsearch.yml 
[root@es-node1 ~]# vim /etc/elasticsearch/jvm.options
```

#### 2.2.4 elasticsearch的日志文件

登录后复制

```plain
[root@es-node1 ~]# tail -f /var/log/elasticsearch/elasticsearch.log
```

#### 2.2.5 启动elasticsearch和加入下次开机自启

登录后复制

```plain
[root@es-node1 ~]# systemctl start elasticsearch
[root@es-node1 ~]# systemctl enable elasticsearch
Created symlink from /etc/systemd/system/multi-user.target.wants/elasticsearch.service to /usr/lib/systemd/system/elasticsearch.service.
```

#### 2.2.6 查看端口

**9200 本地elasticsearch服务本地的端口  
9300 是elasticsearch集群的通信端口**

登录后复制

```plain
[root@es-node1 ~]# netstat -lntup
tcp6       0      0 127.0.0.1:9200          :::*                    LISTEN      2638/java           
tcp6       0      0 ::1:9200                :::*                    LISTEN      2638/java           
tcp6       0      0 127.0.0.1:9300          :::*                    LISTEN      2638/java           
tcp6       0      0 ::1:9300                :::*                    LISTEN      2638/java
```

### 💡2.3操作elasticsearch

#### 2.3.1 操作原理

> ES的操作和我们传统的数据库操作不太一样，它是通过 RestfulAPI 方式进行对ES进行操作，其实本质上就是通过 http的方式去变更我们的资源状态。     
> 通过 URI 的方式指定要操作的资源，比如 Index、Document等。     
> 通过 Http Method 指明资源操作方法，如GET、POST、PUT、DELETE 等。

#### 2.3.2 访问elasticsearch两种方式

*   curl命令本地访问
*   安装kibana访问

#### 2.3.3 安装kibana我们通过访问kibana访问到elasticsearch服务

##### 1> 本地安装kibana

登录后复制

```plain
[root@es-node1 ~]# yum -y localinstall kibana-7.4.0-x86_64.rpm
```

##### 2> 配置kibana

登录后复制

```plain
[root@es-node1 ~]# grep "^[a-Z]" /etc/kibana/kibana.yml
server.port: 5601           #kibana默认监听端口
server.host: "0.0.0.0"      #kibana监听地址段
elasticsearch.hosts: ["http://localhost:9200"]  #kibana丛coordinating节点获取数据
i18n.locale: "zh-CN"        #kibana汉化
```

##### 3>启动kibana

登录后复制

```plain
[root@kibana ~]# systemctl start kibana
[root@kibana ~]# systemctl enable kibana
```

##### 4>查看kibana日志文件

登录后复制

```plain
tail -f /var/log/messages
```

##### 5>查看kibana端口

登录后复制

```plain
[root@es-node1 ~]# netstat -lntup
tcp        0      0 0.0.0.0:5601            0.0.0.0:*               LISTEN      18444/node
```

##### 6>访问kibana网页

![为啥要用EFK日志系统 efk日志收集_elasticsearch_08](https://s2.51cto.com/images/blog/202404/08055250_661315b2dc7a92962.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_elasticsearch_09](https://s2.51cto.com/images/blog/202404/08055251_661315b31e03647013.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 2.3.4通过kibana操作elasticsearch

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_10](https://s2.51cto.com/images/blog/202404/08055251_661315b33543274460.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_elasticsearch_11](https://s2.51cto.com/images/blog/202404/08055251_661315b34895e37019.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

##### 2.3.4.1 操作elasticsearch的索引

###### 1>创建，查看索引

登录后复制

```plain
#创建索引
PUT /cry_index
```

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_12](https://s2.51cto.com/images/blog/202404/08055251_661315b36594551145.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

![为啥要用EFK日志系统 efk日志收集_elasticsearch_13](https://s2.51cto.com/images/blog/202404/08055251_661315b37a4dc44943.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

登录后复制

```plain
#查看所有已存在的索引
GET _cat/indices
```

![为啥要用EFK日志系统 efk日志收集_数据_14](https://s2.51cto.com/images/blog/202404/08055251_661315b390ab34651.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_数据_15](https://s2.51cto.com/images/blog/202404/08055251_661315b39f19060456.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

###### 2>删除索引

登录后复制

```plain
DELETE /cry_index_index
```

![为啥要用EFK日志系统 efk日志收集_java_16](https://s2.51cto.com/images/blog/202404/08055251_661315b3cb7ad25220.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_数据_17](https://s2.51cto.com/images/blog/202404/08055251_661315b3d85e047874.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

##### 2.3.4.2 操作elasticsearch的文档

###### 1> 创建文档

###### ①自定义ID

登录后复制

```plain
#创建一个文档（指定ID）
POST /oldxu_index/_doc/1
{
  "username": "oldxu",
  "age": 18,
  "salary": 1000000
}
```

![为啥要用EFK日志系统 efk日志收集_java_18](https://s2.51cto.com/images/blog/202404/08055252_661315b404bba42645.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

###### ②不指定ID

登录后复制

```plain
#创建一个文档（不指定ID）
POST /oldxu_index/_doc/1
{
  "username": "oldqiang",
  "age": 30,
  "salary": 300
}
```

![为啥要用EFK日志系统 efk日志收集_数据_19](https://s2.51cto.com/images/blog/202404/08055252_661315b41d09985652.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

###### 2> 查询文档

###### ①查询文档，指定要查询的文档id

登录后复制

```plain
GET /oldxu_index/_doc/1
```

![为啥要用EFK日志系统 efk日志收集_elasticsearch_20](https://s2.51cto.com/images/blog/202404/08055252_661315b43e6a286336.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_21](https://s2.51cto.com/images/blog/202404/08055252_661315b452cab76476.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

###### ②查询，搜索所有文档，用\_search

登录后复制

```plain
GET /oldxu_index/_search
```

![为啥要用EFK日志系统 efk日志收集_elasticsearch_22](https://s2.51cto.com/images/blog/202404/08055252_661315b476f8d16074.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_数据_23](https://s2.51cto.com/images/blog/202404/08055252_661315b4aa5ff67693.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

###### 3> 批量创建文档( \_bulk)

**通过 \_bulk 一次创建多个文档，从而减少网络传输开销，提升写入速率。**

登录后复制

```plain
#批量创建document
POST _bulk
{"index":{"_index":"tt","_id":"1"}}
{"name":"oldxu","age":"18"}
{"create":{"_index":"tt","_id":"2"}}
{"name":"oldqiang","age":"30"}
{"delete":{"_index":"tt","_id":"2"}}
{"update":{"_id":"1","_index":"tt"}}
{"doc":{"age":"20"}}
```

![为啥要用EFK日志系统 efk日志收集_elasticsearch_24](https://s2.51cto.com/images/blog/202404/08055252_661315b4bed9d30490.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

###### 4> 批量查询文档

登录后复制

```plain
#批量查询document
GET _mget
{
  "docs": [
    {
      "_index": "tt",
      "_id": "1"
    },
    {
      "_index": "tt",
      "_id": "2"
    }
  ]
}
```

![为啥要用EFK日志系统 efk日志收集_elasticsearch_25](https://s2.51cto.com/images/blog/202404/08055252_661315b4f1a6363796.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

👔03 Elasticsearch集群
--------------------

⚽1.ES集群基本介绍
-----------

### 💡1.1ES集群的好处

**es天然支持集群模式，其好处主要有两个：**

*   **1.能够增大系统的容量，如内存、磁盘，使得 es 集群可以支持PB级的数据；**
*   **2.能够提高系统可用性，即使部分节点停止服务，整个集群依然可以正常服务；**

### 💡1.2 ES如何组集群

**ELasticsearch 集群是由多个节点组成的，通过 cluster.name 设置集群名称，并且用于区分其它的集群，每个节点通过 node.name 指定节点的名称。**

*   **单节点ES，如下图所示；**

![为啥要用EFK日志系统 efk日志收集_java_26](https://s2.51cto.com/images/blog/202404/08055253_661315b52911853239.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

*   **如果单节点出现问题，服务就不可用了，如何新增一个 es 节点加入集群**

![为啥要用EFK日志系统 efk日志收集_数据_27](https://s2.51cto.com/images/blog/202404/08055253_661315b55767618672.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

⚽2.ElasticSearch集群环境部署
----------------------

### 💡2.1 环境准备

<table class="data-table" data-width="" style="outline: none; border-collapse: collapse; width: 100%;"><colgroup><col><col></colgroup><tbody><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>主机名称<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>IP地址<br></p></td></tr><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>es-node1<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>172.16.1.161<br></p></td></tr><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>es-node2<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>172.16.1.162<br></p></td></tr><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>es-node3<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>172.16.1.163<br></p></td></tr></tbody></table>

### 💡2.2 安装ElasticSearch软件

**所有集群节点都需要安装 ElasticSearch软件**

登录后复制

```plain
# yum install java -y
# yum -y localinstall elasticsearch-7.4.0-x86_64.rpm
```

**如果节点内存不够，可以先调制使用的内存大小，默认大小1G**

登录后复制

```plain
vim /etc/elasticsearch/jvm.options
```

![为啥要用EFK日志系统 efk日志收集_java_28](https://s2.51cto.com/images/blog/202404/08055253_661315b575fec43394.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡2.3 node1集群节点配置

#### 2.3.1 恢复环境

**我们在上面这台节点上部署了单节点的配置，现在做集群我们需要清空配置**

登录后复制

```plain
[root@es-node1 ~]#  systemctl stop elasticsearch.service 
[root@es-node1 ~]# rm -rf /var/lib/elasticsearch/*
```

#### 2.3.2 配置三台节点的配置文件

##### node1集群节点配置

登录后复制

```plain
[root@es-node1~]# grep "^[a-Z]" /etc/elasticsearch/elasticsearch.yml
cluster.name: my-app                   #集群名称
node.name: es-node1                     #节点名称
path.data: /var/lib/elasticsearch       #数据存储路径
path.logs: /var/log/elasticsearch       #日志存储路径
#bootstrap.memory_lock: true            #不使用swap分区
network.host: 10.0.0.161                #本机内网IP
http.port: 9200                         #监听端口

discovery.seed_hosts: ["10.0.0.161", "10.0.0.162", "10.0.0.163"]  #集群主机列表
cluster.initial_master_nodes: ["10.0.0.161", "10.0.0.162", "10.0.0.163"]  #仅第一次启动集群时进行选举
```

##### node2集群节点配置

登录后复制

```plain
[root@es-node1~]# grep "^[a-Z]" /etc/elasticsearch/elasticsearch.yml
cluster.name: my-app                   #集群名称
node.name: es-node2                    #节点名称
path.data: /var/lib/elasticsearch       #数据存储路径
path.logs: /var/log/elasticsearch       #日志存储路径
#bootstrap.memory_lock: true            #不使用swap分区
network.host: 10.0.0.162                #本机内网IP
http.port: 9200                         #监听端口

discovery.seed_hosts: ["10.0.0.161", "10.0.0.162", "10.0.0.163"]  #集群主机列表
cluster.initial_master_nodes: ["10.0.0.161", "10.0.0.162", "10.0.0.163"]  #仅第一次启动集群时进行选举
```

##### node3集群节点配置

登录后复制

```plain
[root@es-node1~]# grep "^[a-Z]" /etc/elasticsearch/elasticsearch.yml
cluster.name: my-app                    #集群名称
node.name: es-node3                     #节点名称
path.data: /var/lib/elasticsearch       #数据存储路径
path.logs: /var/log/elasticsearch       #日志存储路径
#bootstrap.memory_lock: true            #不使用swap分区
network.host: 10.0.0.163                #本机内网IP
http.port: 9200                         #监听端口

discovery.seed_hosts: ["10.0.0.161", "10.0.0.162", "10.0.0.163"]  #集群主机列表
cluster.initial_master_nodes: ["10.0.0.161", "10.0.0.162", "10.0.0.163"]  #仅第一次启动集群时进行选举
```

### 💡2.4 三台节点启动elasticsearch,加入下次开机自启

登录后复制

```plain
[root@es-node ~]# systemctl start elasticsearch.service
[root@es-node ~]# systemctl enable elasticsearch
Created symlink from /etc/systemd/system/multi-user.target.wants/elasticsearch.service to /usr/lib/systemd/system/elasticsearch.service.
```

### 💡2.4 ES集群检查

**ES集群检查两种方法**

*   curl命令
*   cerebro软件

#### 2.4.1 curl命令

登录后复制

```plain
[root@es-node1 ~]# curl http://10.0.0.161:9200/_cat/health?v
```

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_29](https://s2.51cto.com/images/blog/202404/08055253_661315b59421c58028.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

登录后复制

```plain
[root@es-node1 ~]# curl http://10.0.0.161:9200/_cluster/health?pretty=true
{
  "cluster_name" : "my-app",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

#### 2.4.2 cerebro软件  [下载地址](https://github.com/lmenezes/cerebro/releases)

##### 1>下载cerebro软件包（安装在任何一个节点上就行）

![为啥要用EFK日志系统 efk日志收集_java_30](https://s2.51cto.com/images/blog/202404/08055253_661315b5af47c64926.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

登录后复制

```plain
[root@es-node1 ~]# wget- o https://github.com/lmenezes/cerebro/releases/download/v0.9.3/cerebro-0.9.3-1.noarch.rpm
```

##### 2>安装cerebro

登录后复制

```plain
yum localinstall -y cerebro-0.9.3-1.noarch.rpm
```

##### 3>配置cerebro

登录后复制

```plain
vim /etc/cerebro/application.conf
data.path = "/tmp/cerebro.db"
```

##### 4>重启cerebro,下次开机自启

登录后复制

```plain
systemctl start cerebro
systemctl enable cerebro
```

##### 5>查看端口

登录后复制

```plain
[root@es-node1 ~]# netstat -lntp
tcp6       0      0 :::9000                 :::*                    LISTEN      19445/java
```

##### 6>网页访问cerebro 10.0.0.161:9000

![为啥要用EFK日志系统 efk日志收集_java_31](https://s2.51cto.com/images/blog/202404/08055253_661315b5c6b7776475.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_数据_32](https://s2.51cto.com/images/blog/202404/08055254_661315b60f3b285561.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_elasticsearch_33](https://s2.51cto.com/images/blog/202404/08055254_661315b62572342657.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_34](https://s2.51cto.com/images/blog/202404/08055254_661315b63ae2e45605.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

**也可以进行索引的创建**

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_35](https://s2.51cto.com/images/blog/202404/08055254_661315b66faf943070.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_36](https://s2.51cto.com/images/blog/202404/08055254_661315b6af73544883.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

⚽3.ES集群节点角色类型
-------------

**es 集群中节点类型介绍**

*   Cluster State
*   Master
*   Data
*   Coordinating

### 💡3.1 Cluster State

**Cluster State：集群相关的数据称为 cluster state；会存储在每个节点中，主要有如下信息:**

*   1）节点信息，比如节点名称、节点连接地址等
*   2）索引信息，比如索引名称、索引配置信息等

### 💡3.2 Master

*   1.ES集群中只能有一个 master 节点，master节点用于控制整个集群的操作；
*   2.master 主要维护 Cluster State，当有新数据产生后，Master 会将最新的数据同步给其他 Node 节点；
*   3.master节点是通过选举产生的，可以通过 node.master: true 指定为Master节点。（ 默认true ）

![为啥要用EFK日志系统 efk日志收集_数据_37](https://s2.51cto.com/images/blog/202404/08055255_661315b71ed0f14504.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

**当我们通过API创建索引 PUT /oldxu\_index，Cluster State 则会发生变化，由 Master 同步至其他 Node 节点；**

![为啥要用EFK日志系统 efk日志收集_数据_38](https://s2.51cto.com/images/blog/202404/08055255_661315b74989482430.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡3.3 Data

*   1.存储数据的节点即为 data 节点，默认节点都是 data 类型，相关配置node.data: true（ 默认为 true ）
*   2.当创建索引后，索引创建的数据会存储至某个节点，能够存储数据的节点，称为data节点；

### 💡3.4 Coordinating

*   1.处理请求的节点即为 coordinating 节点，该节点为所有节点的默认角色，不能取消
*   2.coordinating 节点主要将请求路由到正确的节点处理。比如创建索引的请求会由 coordinating 路由到 master 节点处理；当配置 node.master: false、node.data:false 则为 coordinating 节点

⚽4.ES集群分片副本
-----------

### 💡4.1 提高ES集群可用性

**如何提高 ES 集群系统的可用性；有如下两个方面；**

> 1.服务可用性：  
> 1）2个节点的情况下，允许其中1个节点停止服务；  
> 2）多个节点的情况下，坏的节点不能超过集群一半以上；  
> 2.数据可用性：  
> 1）通过副本 replication 解决，这样每个节点上都有完备的数据。  
> 2）如下图所示，node2上是 oldxu\_index 索引的一个完整副本数据。

![为啥要用EFK日志系统 efk日志收集_java_39](https://s2.51cto.com/images/blog/202404/08055255_661315b79358460172.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡4.2 增大ES集群的容量

> 1.如何增大 ES 集群系统的容量；我们需要想办法将数据均匀分布在所有节点上； 引入分片 shard 解决；  
> 2.什么是分片，将一份完整数据分散为多个分片存储；  
> 2.1 分片是 es 支持 Pb 级数据的基石  
> 2.2 分片存储了索引的部分数据，可以分布在任意节点上  
> 2.3 分片存在主分片和副本分片之分，副本分片主要用来实现数据的高可用  
> 2.4 副本分片的数据由主分片同步，可以有多个，从而提高读取数据的吞吐量 注意：主分片数在索引创建时指定且后续不允许在更改；默认ES7分片数为1个  
> 3.如下图所示：在3个节点的集群中创建 oldxu\_index 索引，指定3个分片，和1个副本；

![为啥要用EFK日志系统 efk日志收集_数据_40](https://s2.51cto.com/images/blog/202404/08055255_661315b7b93d36301.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_java_41](https://s2.51cto.com/images/blog/202404/08055255_661315b7dce532573.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡4.3 增加节点能否提高容量

**问题：目前一共有3个ES节点，如果此时增加一个新节点是否能提高 oldxu\_index 索引数据容量？**

![为啥要用EFK日志系统 efk日志收集_java_42](https://s2.51cto.com/images/blog/202404/08055255_661315b7f0dca17299.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

> **答案：不能，因为 oldxu\_index 只有3个分片，已经分布在3台节点上，那么新增的第四个节点对于 oldxu\_index 而言是无法使用到的。所以也无法带来数据容量的提升；**

### 💡4.4 增加副本能否提高读性能

**问题：目前一共有3个ES节点，如果增加副本数是否能提高 oldxu\_index 的读吞吐量；**

> **答案：不能，因为新增的副本还是会分布在这 node1、node2、node3 这三个节点上的，还是使用了相同的资源，也就意味着有读请求来时，这些请求还是会分配到 node1、node2、node3  
> 上进行处理、也就意味着，还是利用了相同的硬件资源，所以不会提升读取的吞吐量；**

![为啥要用EFK日志系统 efk日志收集_elasticsearch_43](https://s2.51cto.com/images/blog/202404/08055256_661315b82c32757567.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

**问题：如果需要增加读吞吐量性能，应该怎么来做；**

> **答案：增加读吞吐量还是需要添加节点，比如在增加三个节点 node4、node5、node6 那么将原来的 R0、R1、R2 分别迁移至新增的三个节点上，当有读请求来时会被分配 node4、node5、node6，也就意味着有新的  
> CPU、内存、IO，这样就不会在占用 node1、node2、node3 的硬件资源，那么这个时候读吞吐量才会得到真正的提升；**

![为啥要用EFK日志系统 efk日志收集_java_44](https://s2.51cto.com/images/blog/202404/08055256_661315b85ce6e75401.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

⚽5.ES集群健康检查
-----------

### 💡5.1 如何判断集群状态

**Cluster Health 获取集群的健康状态，整个集群状态包括以下三种:**

> **1.green 健康状态，指所有主副分片都正常分配**  
> **2.yellow 指所有主分片都正常分配，但是有副本分片未正常分配**  
> **3.red 有主分片未分配，表示索引不完备，写可能有问题。（但不代表不能存储数据和读取数据）**

### 💡5.2 如何获取集群状态

**我们可以通过 GET \_cluster/health?pretty=true 方式获取集群状态；**

登录后复制

```plain
[root@es-node1-172 ~]# curl  http://172.16.1.162:9200/_cluster/health?pretty=true
{
  "cluster_name" : "my-oldxu",
  "status" : "green",       # 重点关注status一栏
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 33,
  "active_shards" : 66,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

### 💡5.3 Shell脚本检查状态

**通过 Shell 脚本获取集群状态信息；如果出现异常则触发报警邮件；**

登录后复制

```plain
[root@es-node1-172 ~]# curl  http://172.16.1.162:9200/_cluster/health?pretty=true
{
  "cluster_name" : "my-oldxu",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 33,
  "active_shards" : 66,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

#shell检测脚本
# curl -s  http://172.16.1.162:9200/_cluster/health?pretty=true | grep "status" |awk -F '"' '{print $4}'
```

⚽6.ES集群故障转移
-----------

### 💡6.1 什么是故障转移

**所谓故障转移指的是，当集群中有节点发生故障时，这个集群是如何进行自动修复的。  
ES集群目前是由3个节点组成，如下图所示，此时集群状态是 green**

![为啥要用EFK日志系统 efk日志收集_数据_45](https://s2.51cto.com/images/blog/202404/08055256_661315b870e2697999.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡6.2 模拟节点故障

**假设：node1 所在机器宕机导致服务终止，此时集群会如何处理；大体分为三个步骤：**

*   1.重新选举
*   2.主分片调整
*   3.副本分片调整

#### 6.2.1 重新选举

**node2 和 node3 发现 node1 无法响应；一段时间后会发起 master 选举，比如这里选择 node2 为 master 节点；此时集群状态变为 Red 状态；**

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_46](https://s2.51cto.com/images/blog/202404/08055256_661315b886e2045018.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 6.2.2 主分片调整

**node2 发现主分片 P0 未分配，将 node3 上的 R0 提升为主分片；此时所有的主分片都正常分配，集群状态变为 Yellow状态；**

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_47](https://s2.51cto.com/images/blog/202404/08055256_661315b8ac12629746.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 6.2.3 副本分片调整

**node2 将 P0 和 P1 主分片重新生成新的副本分片 R0、R1，此时集群状态变为 Green；**

![为啥要用EFK日志系统 efk日志收集_elasticsearch_48](https://s2.51cto.com/images/blog/202404/08055256_661315b8c385c25119.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡6.3 模拟实验操作

#### 6.3.1 创建一个索引名称为oldxu\_index，3个分片1个副本

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_49](https://s2.51cto.com/images/blog/202404/08055257_661315b91ec9c49774.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

![为啥要用EFK日志系统 efk日志收集_数据_50](https://s2.51cto.com/images/blog/202404/08055257_661315b950fc281973.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

#### 6.3.2 模拟集群中的node1节点损坏

登录后复制

```plain
[root@es-node1 ~]# systemctl stop elasticsearch.service
```

![为啥要用EFK日志系统 efk日志收集_数据_51](https://s2.51cto.com/images/blog/202404/08055257_661315b981e7044745.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_52](https://s2.51cto.com/images/blog/202404/08055257_661315b9bdc8862637.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

⚽7.ES文档路由原理
-----------

**ES文档分布式存储，当一个文档存储至 ES集群时，存储的原理是什么样的?  
如图所示，当我们想一个集群保存文档时，Document1是如何存储到分片P1的？选择P1的依据是什么？**

![为啥要用EFK日志系统 efk日志收集_为啥要用EFK日志系统_53](https://s2.51cto.com/images/blog/202404/08055258_661315ba08ad661858.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

**其实是有一个文档到分片的映射算法，其目是使所有文档均匀分布在所有的分片上，那么是什么算法呢？随机还是轮询呢? 这种是不可取的，因为数据存储后，还需要读取，那这样的话如何读取呢？  
实际上，在ES 中，通过如下的公式计算文档对应的分片存储到哪个节点，计算公式如下:**

登录后复制

```plain
shard = hash(routing) % number_of_primary_shards
# hash                      算法保证将数据均匀分散在分片中
# routing                   是一个关键参数，默认是文档id，也可以自定义。
# number_of_primary_shards  主分片数

# 注意：该算法与主分片数相关，一但确定后便不能更改主分片。
# 因为一旦修改主分片修改后，Share的计算就完全不一样了。
```

### 💡7.1 文档的创建流程

![为啥要用EFK日志系统 efk日志收集_java_54](https://s2.51cto.com/images/blog/202404/08055258_661315ba3f85022680.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡7.2 文档的读取流程

![为啥要用EFK日志系统 efk日志收集_数据_55](https://s2.51cto.com/images/blog/202404/08055258_661315ba59c3320365.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡7.3 文档批量创建的流程

![为啥要用EFK日志系统 efk日志收集_java_56](https://s2.51cto.com/images/blog/202404/08055258_661315ba6af5752131.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

### 💡7.4 文档批量读取的流程

![为啥要用EFK日志系统 efk日志收集_数据_57](https://s2.51cto.com/images/blog/202404/08055258_661315baaaa1974123.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

⚽8.ES扩展集群节点
-----------

### 💡8.1 节点扩展环境准备

在这里我就使用已经拥有的两个节点进行实验

<table class="data-table" data-width="" style="outline: none; border-collapse: collapse; width: 100%;"><colgroup><col><col></colgroup><tbody><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>主机名称<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>IP地址<br></p></td></tr><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>es-node4<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>10.0.0.7<br></p></td></tr><tr style="height: 30px;"><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>es-node5<br></p></td><td style="min-width: auto; overflow-wrap: break-word; margin: 4px 8px; border: 1px solid rgb(217, 217, 217); padding: 4px 8px; cursor: default; vertical-align: top;"><p>10.0.0.8<br></p></td></tr></tbody></table>

### 💡8.2 两台节点安装elasticsearch服务

登录后复制

```plain
yum -y localinstall elasticsearch-7.4.0-x86_64.rpm
```

### 💡8.3 配置两台节点配置文件

**把node4设置为 data角色只负责存储数据** `**node.data: true**` **不参与竞选master** **`node.master:false`**

登录后复制

```plain
[root@web01 ~]# grep "^[a-Z]" /etc/elasticsearch/elasticsearch.yml 
cluster.name: my-app
node.name: node-4
node.master: false
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 10.0.0.7
http.port: 9200
discovery.seed_hosts: ["10.0.0.161", "10.0.0.162"]
```

**把node5设置为路由角色只负责连接** `**node.master: false node.data: false**`

登录后复制

```plain
[root@web02 ~]# grep "^[a-Z]" /etc/elasticsearch/elasticsearch.yml 
cluster.name: my-app
node.name: node-5
node.master: false
node.data: false
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 10.0.0.8
http.port: 9200
discovery.seed_hosts: ["10.0.0.161"]
```

### 💡8.4 启动elasticsearch,加入下次自启

登录后复制

```plain
systemctl start elasticsearch
systemctl enable elasticsearch
```

### 💡8.5 cerebro软件检测新加入的集群的节点

**data角色类型**

![为啥要用EFK日志系统 efk日志收集_数据_58](https://s2.51cto.com/images/blog/202404/08055258_661315bac391915822.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

**Coordinating角色类型**

![为啥要用EFK日志系统 efk日志收集_elasticsearch_59](https://s2.51cto.com/images/blog/202404/08055258_661315badabbd66005.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184 "在这里插入图片描述")

  

整理的一些关于【IT人转技术管理】的项目学习资料（附讲解～～），需要自取：

 [https://d.51cto.com/eDOcp1](https://d.51cto.com/eDOcp1)

本文转自 <https://blog.51cto.com/u_16099245/10666752>，如有侵权，请联系删除。