用gitlab-runner镜像启动gitlab-runner服务

```bash
#获取镜像
docker pull gitlab/gitlab-runner
#启动镜像
docker run \
-dit \
--rm \
--name tool_runner \
--network main_net \
--ip 10.115.0.31 \
-v /data/tool/runner/config:/etc/gitlab-runner \
-v /data/tool/runner/data:/home/gitlab-runner \
gitlab/gitlab-runner
#进入容器内部
docker exec -it tool_runner bash
#生成公私钥对
ssh-keygen
#注册RUNNER参数命令（一容器多注册runner方式）
gitlab-ci-multi-runner register
#以下为提示输入信息
Running in system-mode.                            
#输入你的gitlab访问地址（如果为私网地址，需要在runner容器的hosts文件配置私网地址与域名的关系）
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
https://gitlab.com/
#输入gitlab-ci令牌
Please enter the gitlab-ci token for this runner:
dasfasdfsdadfa
#输入gitlab-ci中关于runner的介绍
Please enter the gitlab-ci description for this runner:
deploy_to_test
#输入gitlab-ci标记（输入如deploy发布，dev测试，test测试，uat等，用于标记runner运行）
Please enter the gitlab-ci tags for this runner (comma separated):
test
#是否允许未标记的构建
Whether to run untagged builds [true/false]:
[false]: 
#是否锁定Runner到当前项目
Whether to lock the Runner to current project [true/false]:
[true]: 
#输入runner脚本执行解释器，此处选择ssh方式
Please enter the executor: shell, ssh, virtualbox, kubernetes, docker, docker-ssh, parallels, docker+machine, docker-ssh+machine:
ssh
#输入需要ssh登陆的服务器地址（实际输入的是容器的IP）
Please enter the SSH server address (e.g. my.server.com):
my.server.com
#输入需要ssh登陆的服务器的ssh端口号，如22
Please enter the SSH server port (e.g. 22):
22
#输入需要ssh登陆的服务器的用户名，如root
Please enter the SSH user (e.g. root):
root
#输入需要ssh登陆的服务器的用户密码。（如果使用公私钥登陆方式可以不输入）
Please enter the SSH password (e.g. docker.io):

#输入需要ssh登陆的认证文件（实际是公私钥文件的私钥文件）
#补充说明：此处配置是为了使用免密登陆。A服务器需要免密登陆B服务器，需要A服务器生成公私钥对，并将公钥信息存在在B服务器的授权公钥文件中。
Please enter path to SSH identity file (e.g. /home/user/.ssh/id_rsa):
/root/.ssh/id_rsa
```