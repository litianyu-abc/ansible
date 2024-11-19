 

  前段时间重新安装了 [Ubuntu](https://so.csdn.net/so/search?q=Ubuntu&spm=1001.2101.3001.7020) 22.04 LTS，安装后没有显示 GRUB 引导页面（默认自动跳过），直接使用默认内核启动，而我需要变更一下默认的内核版本，特此记录一下修改过程。

安装其他版本内核
--------

  Ubuntu 中安装其他版本的内核非常简单，内核其实就是相当于一个软件（DEB 包），安装方式与其他软件并没有啥区别。首先，使用命令 `uname -sr` 就可以查看当前运行的内核版本。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1462cf708d9262a2311a9b0621557182.png)  
  注意，这里说的是安装一个其他版本的内核，与更新当前 Ubuntu 的内核不同。如果是更新当前 Ubuntu 的内核，则是直接使用命令 `sudo apt-cache search linux-image-` 可以搜到针对当前 Ubuntu 版本的官方发布的不同版本的内核，然后使用 `sudo apt-get install xxx` 即可。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/32ec848d6ef108be6a28bacf48fd4218.png)  
  默认情况下，Ubuntu 系统使用 Ubuntu 存储库提供的 Ubuntu 修改过的内核。出于测试、验证等原因，Ubuntu 也提供了各个版本的构建好的 Mainline 内核。而这里我们主要是讲从 https://kernel.ubuntu.com/~kernel-ppa/mainline/ 安装其他版本的 Mainline 内核。

> Ubuntu 内核与主线并不完全相同，https://people.canonical.com/~kernel/info/kernel-version-map.html 给出了对应关系

  _**https://kernel.ubuntu.com/~kernel-ppa/mainline/ 上提供的内核是由未经修改的上游内核（upstream kernel）源代码制成，但使用了 Ubuntu 内核配置文件。**_ 然后，将这些文件打包成 `.deb` 文件，以方便大家使用。

> 1.  这些内核不受支持，不提供任何安全更新
> 2.  如果使用了选定的专有或树外模块（例如 bcmwl、fglrx、NVIDIA 专有图形驱动程序、VirtualBox等），需要先卸载，否则可能导致安装失败
> 3.  官方说明 [MainlineBuilds](https://wiki.ubuntu.com/Kernel/MainlineBuilds)

### 手动安装

  要安装新的 Ubuntu 的内核，首先打开 http://kernel.ubuntu.com/~kernel-ppa/mainline/，然后从列表中选择需要的版本。打开页面就会发现 Ubuntu 官方对该内核针对**不同架构平台**的构建（`xxx/build`）和测试（`xxx/self-tests`）情况，所以，务必注意不要选择错了架构（例如我这里是 amd64）。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2354b4375d3d73a9587b04465a1a0372.png)  
  如上所示，内核并不是只有一个包，通常会有 `linux-headers-*-generic_*`、 `linux-headers-*_all`、 `linux-image-unsigned-*-generic_*`、 `linux-modules-*-generic_*` 四个软件包（我们通常需要的就是名字中带有 `generic` 字样的包），我们可以使用 `dpkg -x xx.deb ./xxx` 将这些包解压看到里面的内容。

1.  `linux-headers-*`：这个包里主要就是我们在应用编程时使用的各种 C 头文件，因为在大版本升级时，这些头文件可能会变化，因此需要替换。  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/80956d9fa995af6f63f70e022578f00a.png)  
    `linux-headers-*`有时会有两个，这主要是因为 Ubuntu 提供了不用用途的内核，带有 `_all` 的那个包含了实际的各种文件，带有 `generic_` 的那个里面通常是一些指向带有 `_all` 的那个里面的符号
    
2.  `linux-image-*`： 这个包里主要就是编译出来的内核镜像文件 `vmlinuz`  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec0a12d10f1648e6adc484a511e3af81.png)
    
3.  `linux-modules-*`：这个包里主要就是那些 `.ko` 驱动  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ffe960b909923d2de7395740f2236450.png)
    

  如下所示，某些内核下面可能还会有一些与 `generic` 对应的 `lowlatency` 的包。`lowlatency` 是用于工业嵌入式系统的低延迟 Linux 内核。官方介绍：https://ubuntu.com/blog/industrial-embedded-systems-iii。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/823bf6c3e22f7e3add68068f6fc61ee8.png)  
安装步骤：

1.  下载各个包
    
    ```c
    wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.19.17/amd64/linux-headers-5.19.17-051917-generic_5.19.17-051917.202210240939_amd64.deb
    wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.19.17/amd64/linux-headers-5.19.17-051917_5.19.17-051917.202210240939_all.deb
    wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.19.17/amd64/linux-image-unsigned-5.19.17-051917-generic_5.19.17-051917.202210240939_amd64.deb
    wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.19.17/amd64/linux-modules-5.19.17-051917-generic_5.19.17-051917.202210240939_amd64.deb
    ```
    
2.  直接使用命令 `sudo dpkg -i *.deb` 安装下载的所有包即可

### Mainline Kernel Installer

  相比手动安装，Ubuntu 下还有个第三方的带 GUI 的内核安装器：Mainline Kernel Installer，我们只需要点点鼠标，其会自动从 http://kernel.ubuntu.com/~kernel-ppa/mainline/ 下载内核的各个包，然后安装。

1.  添加安装源 `sudo add-apt-repository ppa:cappelikan/ppa -y`
2.  安装 `sudo apt install mainline`
3.  从开始界面打开 Mainline Kernel Installer 选择要安装的内核即可  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4382ec80b473acf435cb8dfc48114716.png)

> Mainline Kernel Installer 会自动过滤掉不支持当前架构平台的内核

修改默认版本内核
--------

  在实际工作中，有时候我们需要在 Ubuntu 中添加多个不同版本的内核。所有已安装的可用内核可以在 `/boot/grub/grub.cfg` 这个文件中查看到。`/boot/grub/grub.cfg` 这里面就对应于 [GRUB](https://so.csdn.net/so/search?q=GRUB&spm=1001.2101.3001.7020) 引导页面中的各条目内核的启动参数。

### 命令行方式

1.  首先打开 `sudo nano /boot/grub/grub.cfg`，从中选择要配置的内核参数。_**这个文件就是 GRUB 启动项的菜单描述文件！GRUB 在启动中会读取该文件，然后显示出来就是我们看到的 GRUB 引导界面。**_  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ef6037d3b6432086196e5a7dd91c45c.png)  
      我这里没有安装其他内核，默认的是有 `Linux 5.15.0-53-generic` 和 `Linux 5.15.0-56-generic` 这两个（以及这两个对应的 recovery mode 模式）。其中，我添加的蓝色和红色标号代表不同层级的菜单项的标号。
    
    > 不要试图直接编辑 `/boot/grub/grub.cfg`，这个文件会根据默认配置自动更新
    
2.  编辑默认的 GRUB 配置文件：`sudo nano /etc/default/grub`，其中，默认的 `GRUB_DEFAULT=0` 就表示使用上面的第 0 个菜单项（也就是默认选中 Ubuntu 这条菜单），这里我以修改为 `Linux 5.15.0-53-generic` 为例。  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3774a47eb4db50e5bdf0d56b1c48c587.png)  
      我们只需要将需要的内核对应的菜单添加到 `GRUB_DEFAULT=0` 这一项上即可。**注意，修改的格式是需要包含菜单层级**：`Advanced options for Ubuntu>Ubuntu, with Linux 5.15.0-53-generic` 或者直接简写 `1>2`
    
    > 由于内容中包含了空格，因此必须使用双引号
    
3.  保存后退出。然后执行 `sudo update-grub`，最后 `sudo reboot` 重启应该就可以会自动选择我们指定的内核了。
    

### grub-customizer

  Ubuntu 中有一个名为 `grub-customizer` 的带 GUI 的 GRUB 编辑器，不喜欢使用命令行的可以直接安装这个工具。使用也比较简单直接，简单说一下如何安装，就不过多介绍使用方法了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/204185895ecf0926e286fc65d2c50985.png)

1.  Ubuntu 20.04 版本可以直接使用命令 `sudo apt install grub-customizer` 来安装
2.  Ubuntu 22.04 中由于一个 [BUG](https://bugs.launchpad.net/ubuntu/+source/grub-customizer/+bug/1969353)，官方没有提供该工具，因此需要从第三方安装源进行安装：
    
    ```c
    sudo add-apt-repository ppa:danielrichter2007/grub-customizer 
    sudo apt update
    sudo apt install grub-customizer
    ```
    

### 开启 GRUB 引导页面

  在默认情况下，如果系统只有一个版本的 Ubuntu（或者说只有一个操作系统），GRUB 引导页面是不会显示的。如果需要打开 GRUB 引导页面就需要编辑 Ubuntu 中的 GRUB 配置文件（不喜欢命令行的也可以直接使用上面说的 `grub-customizer`）。

1.  编辑默认的 GRUB 配置文件：`sudo nano /etc/default/grub`  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f56797f092048cb9b3a99e9b3b5f731.png)
    1.  将 `GRUB_TIMEOUT_STYLE=hidden` 这一行前加个 `#` 注释掉或者改为 `GRUB_TIMEOUT_STYLE=menu`
    2.  修改 `GRUB_TIMEOUT=0`，添加一个合适的启动超时时间，**单位是秒**。超时时间内如果无操作，则自动启动。
2.  保存后退出。然后执行 `sudo update-grub`，最后 `sudo reboot` 重启应该就可以看到 GRUB 引导界面了。  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a029504d6f897a0047bd621529e24c2c.png)

  如果只有一个内核，开启 GRUB 引导页面貌似也没啥意义，还额外增加了启动时间。通常，我们在安装了不同内核时才需要开启。当然，在某些情况下，安装内核后，相关工具可能自动就给我开启了！

自编译内核
-----

  主流 Linux 发行版通常不是直接原生使用 Linux Kernel 官方发布的内核，通常会打一些自己的补丁。以 Ubuntu 为例，对于每个发行版本，官方有自己的 Linux Kernl 代码仓库，可以在 https://kernel.ubuntu.com/git/ 找到对应的源码。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b12e0f5ef81e79f27497abebe8ab79e.png)  
  这里的自定义内核我们最好也是直接使用发行方提供的源码，而不是从 Linux Kernel 官方下载。通过 `lsb_release -a | grep Code*` 查看自己的发行版名字，然后选择对应的源码即可。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6de667eb98535731bd7fe34271feb6d8.png)  
  进入下载的源码目录，切换为想要的分支，然后打上上文说的补丁。然后按照如下步骤编译自己的内核即可：

1.  这里我们复制 Ubuntu 自己的配置 `cp /boot/config-$(uname -r) .config`，这样就省了 `make x86_64_defconfig` 和 `make menuconfig` 了。
    
2.  关闭启用模块签名，以便进行编译
    
    ```c
    scripts/config --disable SYSTEM_TRUSTED_KEYS
    scripts/config --disable SYSTEM_REVOCATION_KEYS
    ```
    
3.  配置 `LOCALVERSION` 变量，为自定义内核添加一个标签（格式就是 6.5.0-zcs）：
    
    ```c
    scripts/config --file .config --set-str LOCALVERSION "-zcs"
    ```
    
4.  安装必要的依赖包 `sudo apt install flex bison libncurses5 libncursesw5 libncurses-dev libssl-dev libelf-dev`
    
5.  `sudo make -j$(nproc)` 启动编译
    
6.  `sudo make modules_install` 按照默认那些内核模块
    
7.  `sudo make install` 安装内核
    
8.  `sudo nano /etc/default/grub` 编辑 GRUB2 的配置文件。这里也可以直接配置上默认内核。
    
    ```c
    GRUB_DEFAULT=0
    # GRUB_TIMEOUT_STYLE=hidden
    GRUB_TIMEOUT=10
    ```
    
9.  `sudo update-grub` 然后 `sudo reboot` 就会显示 GRUB 界面，从中选择自己的内核版本即可。
    

远程桌面
----

  Ubuntu 22.04 自带了微软搞的 RDP（Remote Desktop Protocol），并且是默认的远程桌面，因此，我们可以直接使用 Windows 的远程桌面进行连接。同时也带了 VNC 远程桌面，再也不用手动安装各种 VNC 服务端了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f5c23fab994523c3c2ccf2986331fbd3.png)  
  Ubuntu 22.04 中的这套远程桌面其实就来自于最新的 GNOME 中的 `gnome-remote-desktop`。`gnome-remote-desktop` 还有个配套的命令行工具 `grdctl`，通过 `grdctl` 可以直接在 SSH 中来修改上面 GUI 中的配置。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a72ee64f4833028186e9ff82d12f75bd.png)

### 处理锁屏断开问题

  上面这些都不是重点，重点是默认的远程桌面在自动息屏（屏幕变黑）或者手动执行 Lock 锁屏之后就会断开连接，然而，如果把 Ubuntu 22.04 的息屏直接关闭，屏幕就会一直亮着（容易被领导窥屏），再也不会锁屏了（手动锁屏还是会断开远程桌面）。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c19d0cab38663783aca25ce1a3773534.png)  
  Ubuntu 使用的是 GNOME 桌面，GNOME 提供了很多插件，最终，我在 GNOME 插件中发现了 `Allow Locked Remote Desktop` 这个插件，完美解决了上面的问题。在 Ubuntu 下，GNOME 插件有个带 GUI 的管理程序，通过管理程序可以方便安装卸载各种插件：`sudo apt install gnome-shell-extension-manager gnome-shell-extensions`。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/23edb97e0cb4bd275e9246a0eaa283dc.png)

### 处理重启无法连接问题

  还有一种情况，当我们重启电脑后，远程桌面是无法链接的，因为默认用户没有启用自动登录，导致没有可用的账户来进行远程链接。然而，并不是开了自动登录就可以用的，因为权限问题，自动登录后不允许远程操作，这和常用的 Windows 还是有很大差别的。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84cbe5d4b9f0ea2caa689c6538518d3f.png)  
  解决方法就是将修改 Ubuntu 中的秘钥链：Ubuntu 提供了 Passwords and Keys 这个 GUI 程序，打开后修改 Login 的密码为空（直接在 Login 上鼠标右键，输入原密码，新密码留空，确定即可）。以后即使重启电脑，也会自动登录并可以进行远程桌面链接了（与 Windows 体验一模一样）。

包后缀
---

  在 Ubuntu 下使用 `apt` 命令安装软件包时，经常会遇到这些包都有各种各样的后缀。`apt` 命令安装软件包其实就是安装了一些其他软件的依赖（运行时 `.so` 及 `.a` 文件）。注意，包全名的格式通常是 `包名-版本-patch-后缀`，例如：`libusb-1.0-0`。此外，包的版本号与包中运行时（全名格式通常与包差不多）的版本号可能不一致！

*   `无后缀`：不带任何后缀的包就是一个包含软件运行时（`.so` 文件）包，以供其他软件运行时调用。_**当我们运行某个软件时就需要安装其依赖的这种不带后缀的包！**_ 无后缀的包中的运行时是经过版本化的（运行时文件有版本后缀），如下所示：  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d97126e89deae8ac6f628b80d2dbdecd.png)
    
*   `-dev`：带 `-dev` 后缀的包是开发专用的，其中不仅包含运行时（`.so` 及 `.a` 文件），还包含了该软件包对外提供的开发使用的各种头文件。_**当我们要编译某些软件源码时，就需要安装其依赖的带 `-dev` 后缀的包！**_ `-dev` 后缀的包中的运行时通常是没有版本化的（运行时文件没有版本后缀），如下所示  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/861ff96aeb3b6b1999c4e5c3343be4da.png)
    
    > 1.  debian、ubuntu 常用 `-dev` 后缀；red hat、centos、fedora 常用 `-devel` 后缀
    
*   `-utils`：带 `-utils` 后缀的包里面包含了一些该软件额外的命令行工具。
    
*   `-dbg`：带 `-dbg` 后缀的包里面包含调试符号，通常仅供开发人员使用该软件或调试软件的人员使用
    
*   `-doc`：带 `-dbg` 后缀的包里是本软件包的文档
    

The following packages have been kept back
------------------------------------------

  当我们使用 `sudo apt upgrade` 时，经常会遇到 `The following packages have been kept back`，被保留的软件包不会被升级。出现这个文件的很大原因是 Ubuntu 的阶段性更新策略导致的，[从 21.04 开始](https://discourse.ubuntu.com/t/phased-updates-in-apt-in-21-04/20345)，`apt` 命令实现了阶段性更新策略（以前只有更新管理器支持阶段性更新）。

  在分批更新策略之下，更新包会优先推送给部分用户，以供测试及反馈，只有没问题后才会逐步大面积推送。我们可以在 https://ubuntu-archive-team.ubuntu.com/phased-updates.html 这个页面上看到更新的推送情况。

  阶段性更新是由 python3-update-manager 提供的，具体文件是在 `/usr/lib/python3/dist-packages/UpdateManager/Core/UpdateList.py` 中实现的。`apt` 设置与更新管理器设置略有不同：

```c
Update-Manager::Always-Include-Phased-Updates;
APT::Get::Always-Include-Phased-Updates True;

Update-Manager::Never-Include-Phased-Updates;
APT::Get::Never-Include-Phased-Updates True;
```

可以通过告诉 `apt` 忽略阶段性更新来尽早获得更新：`sudo apt -o APT::Get::Always-Include-Phased-Updates=true upgrade`

参考
--

1.  https://linuxhint.com/install-linux-kernel-ubuntu/
2.  https://linuxhint.com/install-upgrade-latest-kernel-ubuntu-22-04/
3.  https://codechacha.com/ja/ubuntu-update-kerenl/
4.  https://askubuntu.com/questions/1246962
5.  https://cuefe.com/12/
6.  https://blog.csdn.net/weixin\_43764974/article/details/135319158

 

![](https://img-blog.csdnimg.cn/f80c24fa002541b89477373f89e63525.png)

技术干货

![](https://g.csdnimg.cn/extension-box/1.1.6/image/weixin.png) 微信公众号 ![](https://g.csdnimg.cn/extension-box/1.1.6/image/ic_move.png)

深入研究，分享最纯粹的技术干货

本文转自 <https://blog.csdn.net/zcshoucsdn/article/details/128188958>，如有侵权，请联系删除。