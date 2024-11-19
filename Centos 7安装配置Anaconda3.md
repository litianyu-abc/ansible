# Centos 7安装配置Anaconda3

#### 说明

1. Anaconda指的是一个开源的Python发行版本，包括了python和很多常见的软件库, 和一个包管理器conda。常见的科学计算类的库都包含在里面了，使得安装比常规python安装要容易。
2. Anaconda是专注于数据分析的Python发行版本，包含了conda、Python等190多个科学包及其依赖项。

#### 第一步，下载Anaconda，如下

1. 打开地址，[https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M&O=D](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fanaconda%2Farchive%2F%3FC%3DM%26O%3DD)
2. 在最近的日期中，选择一个对应自己系统版本的Anaconda3安装包，x86_64表示兼容32位和64位系统。右键复制链接，在linux中使用wget命令进行下载

![img](https://upload-images.jianshu.io/upload_images/16847375-c48d88c83ab22b94.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

##### 此处因为是外网，下载太慢，我直接选择用迅雷下载后再上传到虚拟机。

#### 第二步，安装Anaconda，如下

##### 1. 在下载目录中执行该文件，根据下载的版本不同，名称会有不同

```sh
bash Anaconda3-2021.05-Linux-x86_64.sh
```

##### 2. 接下来会出现一堆的License许可声明，一路回车向下

![img](https://upload-images.jianshu.io/upload_images/16847375-b6dbb6b18f22849e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1188/format/webp)

出现如下文字，输入yes，然后回车

![img](https://upload-images.jianshu.io/upload_images/16847375-21b2abf6ae054ed2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

##### 3. 选择安装目录，如果无需更改直接回车Enter，如需更改要输入绝对路径

```sh
Anaconda3 will now be installed into this location:
/root/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/root/anaconda3] >>> 
```

##### 4. 这里可能会出现错误，提示找不到bunzip2

```sh
Anaconda3-2019.03-Linux-x86_64.sh: line 353: bunzip2: command not found
tar: This does not look like a tar archive
tar: Exiting with failure status due to previous errors
```

##### 5. 使用yum安装，名称是bzip2

```sh
yum install -y bzip2
```

**6. 解决完bzip2后，重复前面的步骤安装anaconda3，等待一会儿后有选项，问是否需要进行conda的初始化，建议输入no。若选择yes，是在/root/.bashrc目录中自动添加环境变量，会使得开机自动启动base环境。**

```sh
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> no
```

##### 7. 看到如下提示则安装成功

```sh
Thank you for installing Anaconda3!

===========================================================================

Anaconda and JetBrains are working together to bring you Anaconda-powered
environments tightly integrated in the PyCharm IDE.

PyCharm for Anaconda is available at:
https://www.anaconda.com/pycharm
```

#### 第三步，配置环境变量

##### 1. 如果conda的初始化时选择了yes，那么此时已经配置好了环境变量，输入以下命令进行验证

```sh
# 进入conda环境 出现(base)则说明安装成功，如图
conda activate
# 退出conda环境
conda deactivate
```

![img](https://upload-images.jianshu.io/upload_images/16847375-b59ba047c6d3d8f4.png?imageMogr2/auto-orient/strip|imageView2/2/w/680/format/webp)



##### 2. 如果conda的初始化时选择了no，则需要自行配置环境变量

###### 1. 打开终端，输入命令，vi /etc/profile，编辑环境变量，添加如下配置，其中的/root/anaconda3/bin/python3/bin，自行替换成自己的安装位置

```sh
# anaconda3 环境变量
PATH=$PATH:$HOME/bin:/root/anaconda3/bin/python3/bin
export PATH
```

**2.保存后记得要刷新一下环境变量**

```sh
source /etc/profile
```





# 在centos上安装Anaconda

介绍：

Anaconda包括Conda、Python以及一大堆安装好的工具包，比如：numpy、pandas等

Miniconda包括Conda、Python

conda是一个开源的包、环境管理器，可以用于在同一个机器上安装不同版本的软件包及其依赖，并能够在不同的环境之间切换

环境：使用3A服务器远程搭建的centos上操作

1、使用wget下载安装包

```shell
wget https://repo.anaconda.com/archive/Anaconda3-5.3.0-Linux-x86_64.sh
```

2、安装anaconda

```shell
chmod +x Anaconda3-5.3.0-Linux-x86_64.sh

./Anaconda3-5.3.0-Linux-x86_64.sh
```

![img](https://img-blog.csdnimg.cn/2d67bc06fc7b499481a9097f19d1007e.png)

3、点击回车键

此时显示Anaconda的信息，并且会出现More，继续按Enter，直到如下图所示

![img](https://img-blog.csdnimg.cn/4ed7ac500d8640a2967fa86c197e8ce4.png)

输入yes

![img](https://img-blog.csdnimg.cn/cb5a295c5c8e4ecc8f5a5951471c29c8.png)

继续点击回车，这段需要等待

![img](https://img-blog.csdnimg.cn/5e73b18e006f49f08f1e38a9540a9991.png)

输入yes

那你需要自己到这个文件夹设置你安装Anaconda路径（比如上面显示我的是）

```shell
/home/wangke/.bashrc

单击进去，在最后一行添加：
export PATH=/home/anaconda3/bin:$PATH
```

需要把之前的那句话给注释掉如下所示：

```shell
# export PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/sbin:/sbin:$PATH

export PATH=/root/anaconda3/bin:$PATH
```

这里只是个示例，具体的还是要看你们自己安装的路径。

然后保存更改，输入下面这句指令：

```shell
source ~/.bashrc
```

1.8 完成安装以及检测是否安装成功

打开新的终端后，进入自己的文件夹目录下，输入anaconda -V（注意a要小写，V要大写），conda -V ,显示版本信息，若显示则表示安装成功。

```shell
root@dev-wyf-react:~/wyf# conda -V

conda 4.5.11
```



## 四、卸载

### 1.删除anaconda文件目录

这里我们先输入

```shell
ls -l
```

查看文件目录，看到有anaconda3的文件夹

之后我们输入

```shell
rm -rf anaconda3
```

删除文件夹即可。 

![img](https://ask.qcloudimg.com/http-save/yehe-9907988/36ef8b2dfefd952a2584aa8d9fa4ba55.png)

 再次查看，可见文件夹已经删除。

### 2.清理.bashrc中的Anaconda路径

 可以用vim打开文件

```shell
vim ~/.bashrc
```

![img](https://pic3.zhimg.com/80/v2-c8eb040a0a2440cf71608be0a640f192_720w.webp)

删除图上部分，保存后

然后删除配置的路径即可 PATH=/yourInstallPath/anaconda3/bin:$PATH

然后退出vim保存，输入下面命令使修改立即生效。

```shell
source ~/.bashrc
```

关闭该终端即可。

























































































































































































































































































































































































































































































































































































































