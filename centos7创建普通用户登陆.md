# centos7创建普通用户登陆

### 1、首先创建一个test用户

```shell
#查看现有的组ID
[root@localhost opt]# cat  /etc/group

#确保您指定的组 ID 不与这些现有的组 ID 冲突。
#创建用户组
groupadd   -g  1001  zjyn
#创建用户
useradd   -u  1001  -g  zjyn  zjyn
#创建用户密码
passwd  zjyn
```

### 2、切换到test用户下

> 当我们执行sudo命令时，会提示输zjyn用户密码，输完后提示，zjyn用户不在sudoer文件里，所以是无法执行sudo命令的

```shell
#su - zjyn
#sudo  mkdir  abc
```

### 3、切换到root用户

> 编辑/etc/sudoers

```shell
#vim  /etc/sudoers

```

> 找到  ## Allow root to run any commands anywhere 这栏，在root下添加 test用户，

```shell
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
zjyn    ALL=(ALL)       ALL
```

> 找到## Same thing without a password一栏，将%wheel ALL=(ALL)前面的#去掉，

+ 更改前：

```shell
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
```

+ 更改后：

```shell
## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL
```

> wq! 保存退出

### 4、切换到zjyn用户下

> 可以使用sudo命令，输入一次密码后，不再需要再每次都输入密码即可进行操作。

```shell
[root@localhost zjyn]# su - zjyn
上一次登录：六 9月  7 00:53:54 CST 2024pts/0 上
[zjyn@localhost ~]$ sudo  mkdir  abc
[sudo] password for  zjyn:
[zjyn@localhost ~]$ sudo  mkdir  abd
[zjyn@localhost ~]$ sudo  mkdir  abf
[zjyn@localhost ~]$ ll
总用量 0
drwxr-xr-x 2 root root 6 9月   7 00:54 abc
drwxr-xr-x 2 root root 6 9月   7 00:54 abd
drwxr-xr-x 2 root root 6 9月   7 00:55 abf
[zjyn@localhost ~]$ 
```

## Linux配置root权限，免密执行sudo命令

> 配置用户具有root权限，方便后期加sudo执行root权限的命令

```shell
[root@localhost zjyn]# vim  /etc/sudoers
```

> 修改/etc/sudoers文件，在%wheel这行下面添加一行
>
> 如下所示：

```shell
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
#zjyn    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software, 
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
zjyn   ALL=(ALL)    NOPASSWD: ALL
```

> 注意：
>
> ​     用户名这一行不要直接放到root行下面，因为所有用户都属于wheel组，你先配置了用户名具有免密功能，但是程序执行到%wheel行时，该功能又被覆盖回需要密码。
>
> 所以用户名要放到%wheel这行下面

### 配置zjyn用户ssh登陆

默认情况下，用户可以通过SSH登录到系统。如果你想限制或允许用户通过SSH登录，请检查SSH配置文件 `/etc/ssh/sshd_config`。

- 确保 `PermitRootLogin` 设置为 `no`（出于安全原因，禁止root用户通过SSH登录）。
- 确保 `AllowUsers` 或 `AllowGroups` 包含新用户的名字，如果已启用这些选项。

```shell
vim  /etc/ssh/sshd_config
# 在配置文件中查找 PermitRootLogin，确保其设置为：
PermitRootLogin no
```

> 如果需要限制允许SSH登录的用户，在配置文件的底部添加：

```shell
AllowUsers zjyn
```

保存文件后，重新启动SSH服务：

```shell
sudo systemctl restart sshd

```









































