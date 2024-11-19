## centos7离线安装python程序

一、安装anaconda3

+ 先在有互联网的环境下，下载anacond3，链接如下:
+ https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M&O=D
+ 下载anaconda3最新的
+ Anaconda3-2024.02-1-Linux-x86_64.sh

```shell
1、传输到服务器上
2、执行安装
bash  Anaconda3-2024.02-1-Linux-x86_64.sh
3、离线安装虚拟环境
conda create --offline --name retpose  python=3.11
conda create --offline --name zjynpose  python=3.11
```

二、下载pip依赖包

```sh
1、先在互联网环境下，使用pip下载依赖包
pip  download  -d /opt/dir/ -r requirements.txt

2、上传依赖包
3、执行离线安装包
pip install --no-index --find-links=/opt/dir/ -r requirements.txt

```

或者打包

```shell
#进入虚拟环境目录
(base) [root@pro-hgh-db ~]# cd  /data/anaconda3/envs/face_recognition/lib/python3.11/
(base) [root@pro-hgh-db python3.11]# pwd
/data/anaconda3/envs/face_recognition/lib/python3.11
#然后打包site-packages
[root@pro-hgh-db python3.11]# tar  czvf  site-packages.tar  site-packages/
```

启动python程序时报

```
ImportError: libGL.so.1: cannot open shared object file: No such file or directory
```

![image-20240826101224429](C:\Users\EDY\AppData\Roaming\Typora\typora-user-images\image-20240826101224429.png)

解决方法：

```shell
遇到 ImportError: libGL.so.1: cannot open shared object file: No such file or directory 错误，通常意味着你的系统缺失了OpenGL库文件，特别是libGL.so.1这个共享库。这个问题在尝试运行依赖于OpenGL的应用程序或库时可能会出现。以下是一些解决此问题的步骤：
对于Linux系统：
1. 安装缺失的库文件： 大多数Linux发行版通过包管理器提供OpenGL库。你可以使用相应的包管理器来安装缺失的库。以下是几种常见Linux发行版的命令示例：
•  Debian/Ubuntu:
sudo apt-get update
sudo apt-get install libgl1-mesa-glx

•  Fedora:
sudo dnf install mesa-libGL

•  CentOS/RHEL:
sudo yum install mesa-libGL
# 或者，如果是CentOS 8+ 使用dnf
sudo dnf install mesa-libGL

•  Arch Linux:
sudo pacman -Syu mesa-libgl

2. 创建符号链接： 如果安装后问题依然存在，可能是由于库文件的版本问题导致的。你可能需要创建一个符号链接来指向已安装的库文件的正确版本。首先，找到实际安装的库文件，例如libGL.so.2，然后创建到libGL.so.1的链接：
sudo ln -s /usr/lib/x86_64-linux-gnu/mesa/libGL.so.2 /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1

注意：路径可能因系统和安装的具体情况而异，请根据实际情况调整。
3. 更新环境变量： 确保库文件的路径包含在LD_LIBRARY_PATH环境变量中。你可以通过以下命令临时添加路径（需要替换为实际路径）：
export LD_LIBRARY_PATH=/path/to/library:$LD_LIBRARY_PATH

若要永久添加，可以编辑~/.bashrc或相应的shell配置文件。
4. 重新启动应用程序或系统： 完成上述步骤后，重启你的应用程序或甚至整个系统，以确保所有更改生效。
对于其他平台（如Windows或macOS）：
这个问题在非Linux系统上不太常见，但如果遇到，可能是因为OpenGL支持未正确配置或缺失。对于这些系统，确保你的开发环境（如Python虚拟环境）正确配置，并且图形驱动及相关的OpenGL支持库已安装和更新。对于macOS，通常系统自带足够的OpenGL支持；对于Windows，可能需要检查与你的开发环境相关的OpenGL库是否正确安装。
如果问题依然存在，查阅具体应用程序或库的官方文档，或寻求社区帮助，可能会提供更多针对性的解决方案。
```

