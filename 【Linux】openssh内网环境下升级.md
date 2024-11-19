 

因公司多数环境处于内网中，无法联网更新，本篇内容主要讲内网更新方法。本篇内容虽经过多次实测，但建议参照本篇升级时先在测试环境进行验证。

所需环境：

本地搭建可连接网络的测试环境，供下载相关的依赖包

#### **一、查看当前机器的版本号**

<table><tbody><tr><td><pre data-index="0" class="set-code-show" name="code" style="user-select: auto;">ssh -V</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/010adb4df1bc4161ae229efb62cec18a.png)

#### 二、了解当前最新openssh与对应兼容的openssl版本信息

[OpenSSHthe main OpenSSH page![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://www.openssh.com/](https://www.openssh.com/ "OpenSSH")

以下可以看到最新的是openssh9.4版本，其对应的openssl版本是 >= 1.1.1


![](https://img-blog.csdnimg.cn/d58694e402124d39892e7a225b22e8b8.png)

#### 三、下载对应的openssh、openssl包

openssh ： [Index of /pub/OpenBSD/OpenSSH/portable/![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/](https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/ "Index of /pub/OpenBSD/OpenSSH/portable/")

openssl ： [/source/old/1.1.1/index.html![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://www.openssl.org/source/old/1.1.1/](https://www.openssl.org/source/old/1.1.1/ "/source/old/1.1.1/index.html")


![](https://img-blog.csdnimg.cn/a41a3b50c6da4a42be41b34c0e038c2e.png)

#### 四、下载openssh相关依赖包

<table><tbody><tr><td><pre data-index="1" class="set-code-show" name="code" style="user-select: auto;">#该命令是下载rpm包到指定目录中，以下命令的意思是下载zlib、zlib-devel、openssl-devel、pam-devel相关包到/usr/local/powersmart/ssl目录下
yum install --downloadonly --downloaddir=/usr/local/powersmart/ssl zlib zlib-devel openssl-devel pam-devel</pre></td></tr></tbody></table>

后面在进行编译openssh时，缺失包就在下载对应的包即可，这里只是列了常用所需的依赖包，如果服务器上没有gcc的话，还需要加上gcc 、gcc++

![](https://img-blog.csdnimg.cn/ebdc6e60f02b4f668ea8febead73d3f3.png)

#### 五、编译安装dropbear（可选）

dropbear是一款轻量级的ssh工具，安装这个工具主要是防止升级openssh的过程中出现问题后，导致我们无法在连接我们的服务器，保险起见建议安装。

##### 5.1 下载dropbear

[Index of /dropbear![icon-default.png?t=N7T8](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)https://matt.ucc.asn.au/dropbear/](https://matt.ucc.asn.au/dropbear/ "Index of /dropbear")

![](https://img-blog.csdnimg.cn/ea97e26ffc2543798f59770a640ee6e6.png)

##### 5.2 解压dropbear文件

<table><tbody><tr><td><pre data-index="2" class="set-code-show" name="code" style="user-select: auto;"> tar jxvf dropbear-2022.82.tar.bz2</pre></td></tr></tbody></table>

##### 5.3 编译安装dropbear

<table><tbody><tr><td><pre data-index="3" class="set-code-show" name="code" style="user-select: auto;">cd dropbear-2022.82
./configure
make  &amp;&amp; make install</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/7a41ce495c894c6ab46ebf5cd9c782eb.png)

![](https://img-blog.csdnimg.cn/c500ac724d6d403bbb5b3a00cd1966bd.png)

##### 5.4 配置dropbear，生成密钥

<table><tbody><tr><td><pre data-index="4" class="set-code-show" name="code" style="user-select: auto;">mkdir /etc/dropbear
/usr/local/bin/dropbearkey -t dss -f /etc/dropbear/dropbear_dss_host_key
/usr/local/bin/dropbearkey -t rsa -s 4096 -f /etc/dropbear/dropbear_rsa_host_key

</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/ae03b954709a44c1a85f7891187802e6.png)

##### 5.5 启动dropbear,注意启动后需要开放2222端口访问权限

<table><tbody><tr><td><pre data-index="5" class="set-code-show" name="code" style="user-select: auto;">/usr/local/sbin/dropbear -p 2222

#要是服务器有防火墙的话要打开，且要保存iptables策略，这里就不做演示了。
iptables -I INPUT -p tcp --dport 2222 -j ACCEPT</pre></td></tr></tbody></table>

##### 5.6 登录验证

![](https://img-blog.csdnimg.cn/830829646931438c964771d6ae60c044.png)

![](https://img-blog.csdnimg.cn/44443cc2b5c3483aa3c867b8f5cadf06.png)

#### 六、编译安装openssl

##### 6.1 解压openssl

<table><tbody><tr><td><pre data-index="6" class="set-code-show" name="code" style="user-select: auto;">tar -zxvf openssl-1.1.1p.tar.gz</pre></td></tr></tbody></table>

##### 6.2 编译openssl

<table><tbody><tr><td><pre data-index="7" class="set-code-show" name="code" style="user-select: auto;">#该命令是编译到/usr/local/openssl_1_1_1_p下，这里建议openssl以其版本号命名，我是1.1.1p版本，所以命名为openssl_1_1_1_p
./config --prefix=/usr/local/openssl_1_1_1_p</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/597e3b00616c4fa890e060fdc4a39d3d.png)

##### 6.3 安装openssl

<table><tbody><tr><td><pre data-index="8" class="set-code-show" name="code" style="user-select: auto;">#编译成功后，执行安装命令
make &amp;&amp; make install</pre></td></tr></tbody></table>

##### 6.4 备份旧版openssl

<table><tbody><tr><td><pre data-index="9" class="set-code-show" name="code" style="user-select: auto;">mv /usr/bin/openssl /usr/bin/openssl.bak</pre></td></tr></tbody></table>

##### 6.5 建立软链接

<table><tbody><tr><td><pre data-index="10" class="set-code-show" name="code" style="user-select: auto;">ln -sf /usr/local/openssl_1_1_1_p/bin/openssl /usr/bin/openssl
echo "/usr/local/openssl_1_1_1_p/lib" &gt;&gt; /etc/ld.so.conf
ldconfig -v</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/5f03fc3cde9d459d94d7f5135053c8ef.png)

##### 6.6 验证版本

<table><tbody><tr><td><pre data-index="11" class="set-code-show" name="code" style="user-select: auto;">openssl version</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/4250fec7c2e240cda7806777cf34ed3e.png)

#### 七、编译安装openssh

##### 7.1 强制安装所有相关rpm依赖包，也可以逐个安装，我这里图省事

<table><tbody><tr><td><pre data-index="12" class="set-code-show" name="code" style="user-select: auto;">rpm -Uvh  *.rpm  --nodeps  --force </pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/851a56580b6441238912bec0841b1cdc.png)

##### 7.2 解压openssh

<table><tbody><tr><td><pre data-index="13" class="set-code-show" name="code" style="user-select: auto;">tar -zxvf openssh-9.4p1.tar.gz</pre></td></tr></tbody></table>

##### 7.3 编译openssh，注意替换自己的ssl的目录，即--with-openssl-includes、--with-openssl-includes两个参数替换

<table><tbody><tr><td><pre data-index="14" class="set-code-show" name="code" style="user-select: auto;">./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-zlib --with-md5-passwords --with-tcp-wrappers --with-openssl-includes=/usr/local/openssl_1_1_1_p/include --with-openssl-includes=/usr/local/openssl_1_1_1_p</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/479eb9c4228449dc89509f58a58ed22b.png)

小tips：使用命令 echo $? 可以检查编译是否异常，0正常否则异常

![](https://img-blog.csdnimg.cn/2984ace77c5a475cb717be5ae9f189e0.png)

##### 7.4 安装openssh

<table><tbody><tr><td><pre data-index="15" class="set-code-show" name="code" style="user-select: auto;">make &amp;&amp; make install</pre></td></tr></tbody></table>

##### 7.5 验证ssh启动是否会存在异常

<table><tbody><tr><td><pre data-index="16" class="set-code-show" name="code" style="user-select: auto;">sshd -T</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/020757f8b3ac4fe9b1fad65abe212fd5.png)

我这里显示有三个文件是没有权限的，对这三个进行授权

<table><tbody><tr><td><pre data-index="17" class="set-code-show" name="code" style="user-select: auto;">chmod 600 /etc/ssh/ssh_host_rsa_key
chmod 600 /etc/ssh/ssh_host_ecdsa_key
chmod 600 /etc/ssh/ssh_host_ed25519_key</pre></td></tr></tbody></table>

授权后在sshd -T 确认是否还有其他问题

![](https://img-blog.csdnimg.cn/714a0a88f8504096a1cc7354560ab0f1.png)

此处查看无报错信息后进行下一步

##### 7.6 配置root登录

<table><tbody><tr><td><pre data-index="18" class="set-code-show" name="code" style="user-select: auto;">cd /etc/ssh/
vi sshd_config
#PermitRootLogin yes   去掉#</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/908d4f07b59b4ffeba4b93ae11aab2b5.png)

![](https://img-blog.csdnimg.cn/f6983c5406634a40b11b78b25145e847.png)

##### 7.7 重启ssh服务

<table><tbody><tr><td><pre data-index="19" class="set-code-show" name="code" style="user-select: auto;">重启
service sshd restart</pre></td></tr></tbody></table>

##### 7.8 重新连接测试验证版本号

<table><tbody><tr><td><pre data-index="20" class="set-code-show" name="code" style="user-select: auto;">ssh -V</pre></td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/511accde27de43a9b1cb920d14035603.png)

本文转自 <https://blog.csdn.net/xwb602625136/article/details/134457479>，如有侵权，请联系删除。