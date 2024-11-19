一、[Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020)用户权限设置实战
------------------------------------------------------------------------------------

设定Jenkins用户权限的目的~团队使用Jenkins  
给不同用户分配权限的好处

### 1、用户权限配置

用户1:Jenkins 管理员:配置Jenkins,创建和更新Job，运行Job，查看日志  
用户2:Jenkins 任务开发:创建和更新Job，运行Job，查看日志  
用户3:Jenkins 使用者:运行Job,查看日志

### 2、用户权限分配

![image.png](https://i-blog.csdnimg.cn/blog_migrate/c8aaef5e170957750ae5bd1ef2f8f2ae.png)  
新建3个用户  
![image.png](https://i-blog.csdnimg.cn/blog_migrate/15af502f513b38e5ca490b6e3c5eb30b.png)  
![image.png](https://i-blog.csdnimg.cn/blog_migrate/5ac3ee520bd1be2454d98c3d0009e5e5.png)  
分配权限  
![image.png](https://i-blog.csdnimg.cn/blog_migrate/b04237d2a327248e3e65fe2415fb12ac.png)

二、Jenkins运行节点配置实战
-----------------

### 1、增加运行节点的好处

增大Jenkins的任务执行能力  
控制不同任务的运行位置  
不同节点之间保持独立的配置

### 2、实战B-1:添加Jenkins运行节点实战

![image.png](https://i-blog.csdnimg.cn/blog_migrate/32d3f8e26b7beb3f0525cc782705215a.png)

#### 1、相关字段说明：

Remote root directory：远程根目录（绝对路径），相当于Jenkins根目录，存放项目的

workspace（有代码下载的话会下载到这里或生成文件等）和[ssh连接](https://so.csdn.net/so/search?q=ssh%E8%BF%9E%E6%8E%A5&spm=1001.2101.3001.7020)工具（比如remoting.jar）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5f7e4479f8a100b2a0622c225a746e03.png)

Launch method：启动方式，如果是要使用ssh登录方式，则选择Launch agents via SSH（需要安装SSH Build Agents插件）

Host：子节点对应服务器的公网IP地址

Credentials：凭证，即SSH登录凭证（登录信息），选择我们前面步骤创建的凭证

Host Key Verification Strategy：主机密钥验证策略，如果是SSH用户密码的凭证进行连接的话，选择“Non verifying Verification Strategy”策略，如果是密钥的方式，则选择"Known hosts file Verification Strategy“策略

Remoting Work directory：远程工作目录（绝对路径），即jenkins子节点的工作路径，存放一些构建日志数据（比如remoting）。如果未设置（为空）的话，则默认使用Remote root directory字段的路径

Number of executors:同步运行的任务数

远程工作目录：必须已经存在

启动方式：Launch agents via SSH

Credentials：  
username：即我们ssh登录远程服务器的用户  
password：ssh登录远程服务器的密码  
![image.png](https://i-blog.csdnimg.cn/blog_migrate/1fa5c851e185188217330bc8a73d02a7.png)

#### 2、SSH连接方式

SSH连接方式，是主节点通过配置的ssh信息（凭证等），通过ssh登录的方式登录到子节点，是主节点主动连接子节点。  
Jenkins节点启动方式默认支持agent代理方式的，如果想要支持SSH，则需要安装SSH Build Agents插件  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a7d216e25ea5b3b95445c1db40b6c98.png)  
需要创建一个子节点服务器的登录凭证，路径：【Manage Jenkins】–>【Manage Credentials】，Domain选择“global”，点击“Add Credentials”添加登录凭证  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dff83b8a366759439c08ccda1a7284af.png)  
凭证的种类有多种，我们先用Username with password的方式  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/559e3843a31d432fe6703fc53afcb7ec.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6becdf28fb4a78af405a2e4f2013a99.png)  
凭证添加完成后，我们去新增节点，路径：【Manage Jenkins】–>【Nodes and Clouds】，点击“New Node”新增节点，然后配置节点  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47838a04f9f4fda1e39c8e941c4af047.png)  
节点配置好后，点击Save按钮进行保存，主节点会自动去连接子节点，我们可以通过子节点的【Log】去查看ssh连接情况：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16b98b04874c583d96fa20f9f65f63ee.png)  
我们也可以在节点列表中查看连接情况：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8bf3f59d1a5dc9b99c1dfa8adf6e9bb7.png)

### 实战B-2:配置不同类型的节点-Python 节点

![image.png](https://i-blog.csdnimg.cn/blog_migrate/43874b7be5599b624f7f85c43b6cf9a7.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/c002f2f564e6d5ae43bc4eacb4678324.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/210ea332aa8e797f348d9a62036f3728.png)

### 实战B-3:配置不同类型的节点-Java节点

![image.png](https://i-blog.csdnimg.cn/blog_migrate/73e45e3dc87c9e466c7bc43576cbd9f0.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/eb6d54c3f44962750c14363020ef321c.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/cd94fc5975dcdeb7c9e1d848ee7250f7.png)

![image.png](https://i-blog.csdnimg.cn/blog_migrate/f8edb0dac275f647c2460050076ee56a.png)

