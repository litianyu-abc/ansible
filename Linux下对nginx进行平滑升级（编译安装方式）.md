 

**目录**

[要求：](#%E8%A6%81%E6%B1%82%EF%BC%9A)

[一、安装编译环境](#t0)

[二、编译安装nginx1.16.1](#t1)

[1、下载nginx1.16.1tar包  官网下载地址：nginx: download](#1%E3%80%81%E4%B8%8B%E8%BD%BDnginx1.16.1tar%E5%8C%85%C2%A0%20%E5%AE%98%E7%BD%91%E4%B8%8B%E8%BD%BD%E5%9C%B0%E5%9D%80%EF%BC%9Anginx%3A%20download)

[2、解压nginx1.16.1包](#2%E3%80%81%E8%A7%A3%E5%8E%8Bnginx1.16.1%E5%8C%85)

[3、进入nginx-1.16.1/目录](#3%E3%80%81%E8%BF%9B%E5%85%A5nginx-1.16.1%2F%E7%9B%AE%E5%BD%95)

[4、进行编译配置](#4%E3%80%81%E8%BF%9B%E8%A1%8C%E7%BC%96%E8%AF%91%E9%85%8D%E7%BD%AE)

[5、编译安装](#5%E3%80%81%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85)

[6、给nginx配置一个systemctl管理（配置的路径要与nginx安装的路径一致）](#6%E3%80%81%E7%BB%99nginx%E9%85%8D%E7%BD%AE%E4%B8%80%E4%B8%AAsystemctl%E7%AE%A1%E7%90%86%EF%BC%88%E9%85%8D%E7%BD%AE%E7%9A%84%E8%B7%AF%E5%BE%84%E8%A6%81%E4%B8%8Enginx%E5%AE%89%E8%A3%85%E7%9A%84%E8%B7%AF%E5%BE%84%E4%B8%80%E8%87%B4%EF%BC%89)

[7、启动nginx](#7%E3%80%81%E5%90%AF%E5%8A%A8nginx)

[二、安装nginx1.24.0](#t2)

[1、下载nginx-1.24.0，解压并进入nginx-1.24.0目录](#1%E3%80%81%E4%B8%8B%E8%BD%BDnginx-1.24.0%EF%BC%8C%E8%A7%A3%E5%8E%8B%E5%B9%B6%E8%BF%9B%E5%85%A5nginx-1.24.0%E7%9B%AE%E5%BD%95)

[2、将nginx-1.16.1配置的编译文件复制](#2%E3%80%81%E5%B0%86nginx-1.16.1%E9%85%8D%E7%BD%AE%E7%9A%84%E7%BC%96%E8%AF%91%E6%96%87%E4%BB%B6%E5%A4%8D%E5%88%B6)

[3、执行make，不要make install](#3%E3%80%81%E6%89%A7%E8%A1%8Cmake%EF%BC%8C%E4%B8%8D%E8%A6%81make%20install)

[4、移动当前版本的nginx文件](#4%E3%80%81%E7%A7%BB%E5%8A%A8%E5%BD%93%E5%89%8D%E7%89%88%E6%9C%AC%E7%9A%84nginx%E6%96%87%E4%BB%B6)

[5、将nginx-1.24.0目录下的nginx放到/usr/local/nginx/sbin/目录下](#5%E3%80%81%E5%B0%86nginx-1.24.0%E7%9B%AE%E5%BD%95%E4%B8%8B%E7%9A%84nginx%E6%94%BE%E5%88%B0%2Fusr%2Flocal%2Fnginx%2Fsbin%2F%E7%9B%AE%E5%BD%95%E4%B8%8B)

[6、生成新的nginx  master进程](#6%E3%80%81%E7%94%9F%E6%88%90%E6%96%B0%E7%9A%84nginx%C2%A0%20master%E8%BF%9B%E7%A8%8B)

[7、平滑停止旧版本的nginx进程](#7%E3%80%81%E5%B9%B3%E6%BB%91%E5%81%9C%E6%AD%A2%E6%97%A7%E7%89%88%E6%9C%AC%E7%9A%84nginx%E8%BF%9B%E7%A8%8B)

[8、可以看到当前版本已经升级到了1.24.0](#8%E3%80%81%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E5%BD%93%E5%89%8D%E7%89%88%E6%9C%AC%E5%B7%B2%E7%BB%8F%E5%8D%87%E7%BA%A7%E5%88%B0%E4%BA%861.24.0)

[三、升级后出现的问题](#t3)

#### 要求：

不停止业务运行，不能影响用户体验，将nginx1.16.1升级为nginx1.24.0。

#### 一、安装编译环境

```shell
yum install epel-release         //安装epel扩展源
yum install gcc gcc-c++ make zlib-devel pcre pcre-devel openssl-devel perl-devel perl-ExtUtils-Embed gd-devel geoip-devel -y    //安装编译基础环境
```

#### 二、编译[安装nginx](https://so.csdn.net/so/search?q=%E5%AE%89%E8%A3%85nginx&spm=1001.2101.3001.7020)1.16.1

##### 1、下载nginx1.16.1tar包  官网下载地址：[nginx: download](https://nginx.org/en/download.html "nginx: download")

##### 2、解压nginx1.16.1包

```bash
tar xf nginx-1.16.1.tar.gz
```

##### 3、进入nginx-1.16.1/目录

```bash
cd nginx-1.16.1/
```

##### 4、进行编译配置

```bash
 ./configure --prefix=/usr/local/nginx \
--user=nginx \
--group=nginx \
--with-pcre \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_image_filter_module \
--with-http_slice_module \
--with-mail \
--with-threads \
--with-file-aio \
--with-stream \
--with-mail_ssl_module \
--with-stream_ssl_module \
--with-http_geoip_module          \
--with-http_geoip_module=dynamic  \
--with-http_sub_module            \
--with-http_dav_module            \
--with-http_flv_module            \
--with-http_mp4_module            \
--with-http_gunzip_module         \
--with-http_gzip_static_module    \
--with-http_auth_request_module   \
--with-http_random_index_module   \
--with-http_secure_link_module    \
--with-http_degradation_module    \
--with-http_slice_module          \
--with-http_stub_status_module
```

##### 5、编译安装

```bash
make && make install
```

##### 6、给[nginx配置](https://so.csdn.net/so/search?q=nginx%E9%85%8D%E7%BD%AE&spm=1001.2101.3001.7020)一个systemctl管理（配置的路径要与nginx安装的路径一致）

```bash
vim /usr/lib/systemd/system/nginx.service
 
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -S reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```

##### 7、启动nginx

```bash
systemctl start nginx
```

ps：如果启动失败，大概率是因为没有创建nginx用户

创建用户nginx,然后就可以正常启动nginx了

```bash
useradd -s /sbin/nologin nginx
```

#### 二、安装nginx1.24.0

##### 1、下载nginx-1.24.0，解压并进入nginx-1.24.0目录

```bash
tar xf nginx-1.24.0.tar.gz
cd nginx-1.24.0/
```

##### 2、将nginx-1.16.1配置的编译文件复制

```html
[root@web03 nginx-1.24.0]# ./configure --prefix=/usr/local/nginx \
> --user=nginx \
> --group=nginx \
> --with-pcre \
> --with-http_ssl_module \
> --with-http_v2_module \
> --with-http_realip_module \
> --with-http_addition_module \
> --with-http_sub_module \
> --with-http_dav_module \
> --with-http_flv_module \
> --with-http_mp4_module \
> --with-http_gunzip_module \
> --with-http_gzip_static_module \
> --with-http_random_index_module \
> --with-http_secure_link_module \
> --with-http_stub_status_module \
> --with-http_auth_request_module \
> --with-http_image_filter_module \
> --with-http_slice_module \
> --with-mail \
> --with-threads \
> --with-file-aio \
> --with-stream \
> --with-mail_ssl_module \
> --with-stream_ssl_module \
> --with-http_geoip_module          \
> --with-http_geoip_module=dynamic  \
> --with-http_sub_module            \
> --with-http_dav_module            \
> --with-http_flv_module            \
> --with-http_mp4_module            \
> --with-http_gunzip_module         \
> --with-http_gzip_static_module    \
> --with-http_auth_request_module   \
> --with-http_random_index_module   \
> --with-http_secure_link_module    \
> --with-http_degradation_module    \
> --with-http_slice_module          \
> --with-http_stub_status_module
```

##### 3、执行make，不要make install

```html
[root@web03 nginx-1.24.0]# make
```

##### 4、移动当前版本的nginx文件

```html
[root@web03 ~]# cd /usr/local/nginx/sbin/
[root@web03 sbin]# ls
nginx
[root@web03 sbin]# mv nginx /root/
```

##### 5、将nginx-1.24.0目录下的nginx放到/usr/local/nginx/sbin/目录下

```html
[root@web03 sbin]# mv /root/nginx-1.24.0/objs/nginx /usr/local/nginx/sbin/
[root@web03 sbin]# ls
nginx
[root@web03 sbin]#
```

##### 6、生成新的nginx  master进程

```html
[root@web03 ~]# cd /usr/local/nginx/logs/
[root@web03 logs]# cat nginx.pid
15210
[root@web03 logs]# kill -USR2 15210
[root@web03 logs]# ls
access.log  error.log  nginx.pid  nginx.pid.oldbin
[root@web03 logs]#
```

可以看到，当前的nginx进程文件自动命名为nginx.pid.oldbin，又生成了一个新的nginx.pid文件

##### 7、平滑停止旧版本的nginx进程

```html
[root@web03 logs]# cat nginx.pid.oldbin
15210
[root@web03 logs]# kill -WINCH 15210
[root@web03 logs]# kill -QUIT 15210
[root@web03 logs]# ls
access.log  error.log  nginx.pid
[root@web03 logs]#
```

旧版本的nginx进程已经被新版本的nginx进程替代

##### 8、可以看到当前版本已经升级到了1.24.0

```html
[root@web03 logs]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.24.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --user=nginx --group=nginx --with-pcre --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-http_image_filter_module --with-http_slice_module --with-mail --with-threads --with-file-aio --with-stream --with-mail_ssl_module --with-stream_ssl_module --with-http_geoip_module --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module
[root@web03 logs]#
```

9、访问页面测试：输入本机IP

#### 三、升级后出现的问题

1、如果升级之后执行systemctl restart nginx 重启失败，可能是因为配置systemctl启动nginx文件有问题

```html
vim /usr/lib/systemd/system/nginx.service
```

修改配置文件中ExecReload=/usr/local/nginx/sbin/nginx -S reload 将-S改为-s

![](https://i-blog.csdnimg.cn/blog_migrate/b20bc8f7ac10e073ebc4891f6b2a5fa1.png)

再次systemctl restart nginx重启就没问题了，搞定！！！！

```sh
./configure --prefix=/usr/local/nginx \
--user=nginx  \
--group=nginx \
--with-openssl=/usr/local/src/openssl \
--sbin-path=/usr/local/nginx/sbin/nginx \
--modules-path=/usr/local/nginx/modules \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--pid-path=/usr/local/nginx/logs/nginx.pid \
--lock-path=/usr/local/nginx/logs/nginx.lock \
--with-http_gzip_static_module \
--with-http_ssl_module \
--with-stream
```

