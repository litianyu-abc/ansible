# 搭建Loki、Promtail、Grafana轻量级日志系统（centos7） 

# Loki简介

Loki 是一个水平可扩展，高可用性，多租户日志聚合系统,灵感来自 Prometheus ，其设计非常经济高效，易于操作。它不索引日志的内容，而是为每个日志流设置一组标签。

Loki与其他日志聚合系统差别：

- 不对日志进行全文本索引。通过存储压缩的，非结构化的日志以及仅索引元数据，Loki更加易于操作且运行成本更低。
- 使用与Prometheus相同的标签对日志流进行索引和分组，从而使您能够使用与Prometheus相同的标签在指标和日志之间无缝切换。
- 特别适合存储Kubernetes Pod日志。诸如Pod标签之类的元数据会自动被抓取并建立索引。
- 在Grafana中原生支持（需要Grafana v6.0及以上）。

Loki的日志系统的组件：

- Promtail是代理，负责收集日志并将其发送给Loki。
- Loki是主服务器，负责存储日志和处理查询。
- Grafana用于查询和显示日志。

# 搭建步骤

本文采用的搭建方式是分别下载各个组件并安装。也可以参考官方的文档进行搭建安装。

Loki的GitHub地址：https://github.com/grafana/loki 

配置文件官网地址：https://grafana.com/docs/loki/latest/installation/local/ 

Grafana下载官网：https://grafana.com/grafana/download

![img](https://img-blog.csdnimg.cn/20201113175106381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

**1.下载安装启动Grafana**

官网提供了下图中几种方式，本文采用的是CentOS系统，yum安装的方式。

![img](https://img-blog.csdnimg.cn/20201113175518406.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

```shell
#下载安装grafana命令，下列命令执行成功后。在/usr/sbin文件夹下会有grafana-server执行文件
wget https://dl.grafana.com/oss/release/grafana-7.3.2-1.x86_64.rpm
sudo yum install grafana-7.3.2-1.x86_64.rpm
#启动grafana，grafana会占用服务器3000端口，记得保证3000端口不被占用
cd /usr/sbin
./grafana-server web
```

**2.下载启动Loki和Promtail**

官方文档地址：https://grafana.com/docs/loki/latest/installation/local/

因为采用本地安装的方式，参考文档（下图箭头指向的位置），分别下载执行文件和启动的配置文件。

![img](https://img-blog.csdnimg.cn/20201113180855912.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

> 下载Promtail：https://github.com/grafana/loki/releases/download/v2.0.0/promtail-linux-amd64.zip 

```shell
#下载压缩文件
curl -O -L "https://github.com/grafana/loki/releases/download/v2.0.0/loki-linux-amd64.zip"
#解压文件
unzip "loki-linux-amd64.zip"
#执行文件授权
chmod a+x "loki-linux-amd64"
 
#下载Loki和Promtail的配置文件
wget https://raw.githubusercontent.com/grafana/loki/master/cmd/loki/loki-local-config.yaml
wget https://raw.githubusercontent.com/grafana/loki/master/cmd/promtail/promtail-local-config.yaml
```

启动Loki，本文采用的Loki默认配置，服务端口为3100

```shell
#启动Loki命令
nohup ./loki-linux-amd64 -config.file=loki-local-config.yaml  > loki.log 2>&1 &
#查看启动是否成功(查看3100端口的进程是否存在)
netstat -tunlp | grep 3100
#或者根据名称查找进程(执行命令后有下边的显示，则启动成功)
ps -ef | grep loki-linux-amd64
$ root     11037 22022  0 15:44 pts/0    00:00:55 ./loki-linux-amd64 -config.file=loki-local-config.yaml
```

到收集日志的服务器上配置Promtail并启动，传输文件到收集日志的服务器。

**修改配置文件**

![img](https://img-blog.csdnimg.cn/20201113182757304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

**启动Promtail**

```shell
#Promtail默认端口是9080，启动完成后，可以采用上边的方式查看进程是否启动成功
nohup ./promtail-linux-amd64 -config.file=promtail-local-config.yaml > promtail.log 2>&1 &
```

**3.添加数据源**

访问web页面：http://localhost:3000/ 进行登录（账号密码都是admin），点击下图中的位置，找到Loki，配置数据源。

![img](https://img-blog.csdnimg.cn/20201113183321179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

填写数据源的访问地址并保存。

![img](https://img-blog.csdnimg.cn/20201113183459410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

配置好数据源之后就可以点击下图中的位置，进行日志查看了。

![img](https://img-blog.csdnimg.cn/20201113183643442.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)

日志查看效果如下图。

![img](https://img-blog.csdnimg.cn/20201113184320904.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RjeTE0Mjk5MjA2Mjc=,size_16,color_FFFFFF,t_70)















































































































































































































































































































