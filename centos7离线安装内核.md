# centos7离线升级内核

### 下载rpm包

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-6.9.6-1.el7.elrepo.x86_64.rpm
wget https://mirrors.tuna.tsinghua.edu.cn/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-6.9.6-1.el7.elrepo.x86_64.rpm
wget https://mirrors.tuna.tsinghua.edu.cn/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-headers-6.9.6-1.el7.elrepo.x86_64.rpm
```

### 如果缺少perl还需要单独安装

#### 下载perl及其依赖

在其他联网的机器上执行

```shell
yumdownloader --resolve --destdir=. perl

```

#### 传输下载的包到离线的 CentOS 服务器

```shell
scp ~/perl-packages/*.rpm user@server_ip:/tmp

```

#### 在离线的 CentOS 服务器上安装 perl 包及其依赖

```shell
cd /tmp
sudo yum localinstall -y perl*.rpm

```

### 卸载现有的 kernel-headers

```shell
#确保没有就行
 yum remove -y kernel-headers

```

### 安装新版内核

```shell
 yum localinstall -y kernel-ml-*.rpm

```

### 更新GRUB配置

```shell
#安装完成后，需要更新GRUB配置以使系统在启动时使用新内核：
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

```

### 设置默认内核：

```shell
#确保系统默认使用新安装的内核。在CentOS 7上，可以通过以下命令来查看可用的内核版本：
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

找到你安装的6.9.6内核对应的序号，并设置为默认内核：

```shell
sudo grub2-set-default X

```

将X替换为6.9.6内核对应的序号。

### 重启系统

最后，重启系统使更改生效：

```shell
sudo reboot

```

### 验证内核版本：

系统重启后，使用以下命令验证当前运行的内核版本是否为6.9.6：

```shell
uname -r

```

脚本化过程

```shell
#!/bin/bash

RED='\033[1;31m'
NC='\033[0m' # 没有颜色（重置颜色）
current_dir=$(pwd)

# 检查当前系统是否是 CentOS
check_centos() {
  if ! grep -q "CentOS" /etc/os-release; then
    echo -e "${RED}当前系统不是 CentOS，脚本退出。${NC}"
    exit 1
  else
    echo -e "${RED}当前系统是 CentOS，继续执行...${NC}"
  fi
}

# 检查并安装 perl 包及其依赖
check_and_install_perl() {
  if ! rpm -q perl > /dev/null 2>&1; then
    echo -e "${RED}perl 包未安装，正在安装 perl 及其依赖...${NC}"
    cd $current_dir/perl
    sudo yum --disablerepo="*" localinstall -y perl*.rpm > /dev/null 2>&1
    cd $current_dir
    if [ $? -ne 0 ]; then
      echo -e "${RED}安装 perl 包及其依赖失败，请检查传输的包是否完整！${NC}"
      exit 1
    fi
  else
    echo -e "${RED}perl 包已安装，跳过安装步骤。${NC}"
  fi
}

# 卸载现有的 kernel-headers
remove_existing_kernel_headers() {
  if rpm -q kernel-headers > /dev/null 2>&1; then
    echo -e "${RED}正在卸载现有的 kernel-headers 包...${NC}"
    sudo yum remove -y kernel-headers > /dev/null 2>&1
    if [ $? -ne 0 ]; then
      echo -e "${RED}卸载现有的 kernel-headers 包失败！${NC}"
      exit 1
    fi
  else
    echo -e "${RED}没有安装 kernel-headers 包，跳过卸载步骤。${NC}"
  fi
}

# 安装内核包
install_kernel() {
  echo -e "${RED}正在安装内核包...${NC}"
  cd $current_dir/kernel
  sudo yum --disablerepo="*" localinstall -y kernel-ml*.rpm > /dev/null 2>&1
  cd $current_dir
  if [ $? -ne 0 ]; then
    echo -e "${RED}安装内核包失败，请检查传输的包是否完整！${NC}"
    exit 1
  fi
}

# 更新GRUB配置
update_grub() {
  echo -e "${RED}正在更新GRUB配置...${NC}"
  sudo grub2-mkconfig -o /boot/grub2/grub.cfg > /dev/null 2>&1
  if [ $? -ne 0 ]; then
    echo -e "${RED}更新GRUB配置失败！${NC}"
    exit 1
  fi
}

# 设置默认内核
set_default_kernel() {
  echo -e "${RED}正在设置默认内核...${NC}"
  sleep 2
  kernel_version=$(awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg | grep 6.9.6 | cut -d ' ' -f 1)
  if [ -z "$kernel_version" ]; then
    echo -e "${RED}未找到6.9.6版本内核，请检查内核安装是否成功！${NC}"
    exit 1
  fi
  sudo grub2-set-default $kernel_version > /dev/null 2>&1
  if [ $? -ne 0 ]; then
    echo -e "${RED}设置默认内核失败！${NC}"
    exit 1
  fi
}

# 主程序执行步骤
main() {

  check_and_install_perl
  remove_existing_kernel_headers
  install_kernel
  update_grub
  set_default_kernel

  echo -e "${RED}需要重启服务器！！！${NC}"
}

# 执行主程序
main



```





































































































































































































































