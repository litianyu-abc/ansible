# Linux解决用户密码过期但不用修改密码的方法

linux 如果你设置了密码策略（密码有效期设定），到期不修改密码，系统在你登陆的时候会要求你更改密码，不更改便不让你登陆。如果你想不修改密码，延长你的账户有效期的话，可以用chane这个命令。

> chage

```shell
参数意思：

-m 密码可更改的最小天数。为零时代表任何时候都可以更改密码。

-M 密码保持有效的最大天数。

-W 用户密码到期前，提前收到警告信息的天数。

-E 帐号到期的日期。过了这天，此帐号将不可用。

-d 上一次更改的日期

-i 停滞时期。如果一个密码已过期这些天，那么此帐号将不可用。

-l 例出当前的设置。由非特权用户来确定他们的密码或帐号何时过期。
```

例：

```shell
HostNameA:~ # chage -l a
Minimum:        0
Maximum:        90
Warning:        14
Inactive:       -1
Last Change:            Dec 06, 2011
Password Expires:       Mar 05, 2012
Password Inactive:      Never
Account Expires:        Never
HostNameA:~ #
```

![image-20240903094327355](C:\Users\EDY\AppData\Roaming\Typora\typora-user-images\image-20240903094327355.png)

> a账号密码已经在Mar 05,2012过期，现在要延长该帐户密码有效期，可以用chage命令修改上次密码修改时间

修改：

```shell
HostNameA:~ # chage a
Changing aging information for a.
        Minimum Password Age [0]:  #（输入回车）
        Maximum Password Age [90]: #（输入回车）
        Password Expiration Warning [14]: #（输入回车）
        Password Inactive [-1]: #（输入回车）
        Last Password Change (YYYY-MM-DD) [2011-12-06]: 2012-06-29    #（输入当天日期）
        Account Expiration Date (YYYY-MM-DD) [1969-12-31]: #（输入回车）
Aging information changed.
```

查看

```shell
HostNameA:~ #

有效期已经增大了90天

HostNameA:~ # chage -l a
Minimum:        0
Maximum:        90
Warning:        14
Inactive:       -1
Last Change:            Jun 29, 2012
Password Expires:       Sep 27, 2012
Password Inactive:      Never
Account Expires:        Never
HostNameA:~ #
```

## 取消密码有效期，改成永不过期

```shell
[root@localhost ~]# chage -M 99999 -W 7 admin
[root@localhost ~]# chage -l admin
Last password change                    : Dec 02, 2022
Password expires                    : never
Password inactive                   : never
Account expires                     : never
Minimum number of days between password change      : 0
Maximum number of days between password change      : 99999
Number of days of warning before password expires   : 7 
```

