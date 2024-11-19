**1\. 基本说明**

 　　本文章将演示CentOS 8二进制方式安装高可用k8s 1.17.x，相对于其他版本，二进制安装方式并无太大区别。

**2\. 基本环境配置**

　　主机信息

192.168.1.19 k8s-master01
192.168.1.18 k8s-master02
192.168.1.20 k8s-master03
192.168.1.88 k8s-master-lb
192.168.1.21 k8s-node01
192.168.1.22 k8s-node02

　　系统环境

\[root@k8s-master01 ~\]# cat /etc/redhat-release 
CentOS Linux release 8.0.1905 (Core) 
\[root@k8s-master01 ~\]# uname -a
Linux k8s-master01 4.18.0-80.el8.x86\_64 #1 SMP Tue Jun 4 09:19:46 UTC 2019 x86\_64 x86\_64 x86\_64 GNU/Linux

 **k8s全栈架构师视频课程**

    51cto: [https://edu.51cto.com/sd/518e5](https://edu.51cto.com/sd/518e5)  
    腾讯课堂: [https://ke.qq.com/course/2738602](https://ke.qq.com/course/2738602)

　　购买前咨询QQ727585266领取大礼包

　　配置所有节点hosts文件

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 ~\]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.19 k8s-master01
192.168.1.18 k8s-master02
192.168.1.20 k8s-master03
192.168.1.88 k8s-master-lb
192.168.1.21 k8s-node01
192.168.1.22 k8s-node02

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有节点关闭firewalld 、dnsmasq、selinux

systemctl disable --now firewalld 
systemctl disable --now dnsmasq  
setenforce 0

　　所有节点关闭swap分区

\[root@k8s-master01 ~\]# swapoff -a && sysctl -w vm.swappiness=0
vm.swappiness = 0

　　所有节点同步时间

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' /etc/timezone
ntpdate time2.aliyun.com

　　Master01节点生成ssh key

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 ~\]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id\_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id\_rsa.
Your public key has been saved in /root/.ssh/id\_rsa.pub.
The key fingerprint is:
SHA256:6uz2kI+jcMJIUQWKqRcDRbvpVxhCW3Tmqn0NKS+lT3U root@k8s-master01
The key's randomart image is:
+---\[RSA 2048\]----+
|.o++=.o          |
|.+o+ +           |
|oo\* . .          |
|. .\* + .         |
|..+ + = S E      |
|.= o \* \* .       |
|. \* \* B .        |
|   = Bo+         |
|    .+\*oo        |
+----\[SHA256\]-----+

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　Master01配置免密码登录其他节点

\[root@k8s-master01 ~\]# for i in k8s-master01 k8s-master02 k8s-master03 k8s-node01 k8s-node02;do ssh-copy-id -i .ssh/id\_rsa.pub $i;done

　　所有节点安装基本工具

yum install wget jq psmisc vim net-tools yum-utils device-mapper-persistent-data lvm2 git -y

 　　Master01下载安装文件

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 ~\]# git clone https://github.com/dotbalo/k8s-ha-install.git
Cloning into 'k8s-ha-install'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (11/11), done.
remote: Total 461 (delta 2), reused 5 (delta 1), pack-reused 449
Receiving objects: 100% (461/461), 19.52 MiB | 4.04 MiB/s, done.
Resolving deltas: 100% (163/163), done.

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　切换到1.17.x分支

git checkout manual-installation-v1.17.x

**3\. 基本组件安装**

 　　所有节点安装ipvs

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

yum install ipvsadm ipset sysstat conntrack libseccomp -y

\[root@k8s\-master02 ~\]# cat /etc/modules-load.d/ipvs.conf 
ip\_vs
ip\_vs\_lc
ip\_vs\_wlc
ip\_vs\_rr
ip\_vs\_wrr
ip\_vs\_lblc
ip\_vs\_lblcr
ip\_vs\_dh
ip\_vs\_sh
ip\_vs\_fo
ip\_vs\_nq
ip\_vs\_sed
ip\_vs\_ftp

\[root@k8s\-master02 ~\]# systemctl enable --now systemd-modules-load.service

\[root@k8s\-master02 ~\]# lsmod |grep ip\_vs
ip\_vs\_ftp              16384  0
ip\_vs\_sed              16384  0
ip\_vs\_nq               16384  0
ip\_vs\_fo               16384  0
ip\_vs\_dh               16384  0
ip\_vs\_lblcr            16384  0
ip\_vs\_lblc             16384  0
ip\_vs\_wlc              16384  0
ip\_vs\_lc               16384  0
ip\_vs\_sh               16384  0
ip\_vs\_wrr              16384  0
ip\_vs\_rr               16384  11
ip\_vs                 172032  35 ip\_vs\_wlc,ip\_vs\_rr,ip\_vs\_dh,ip\_vs\_lblcr,ip\_vs\_sh,ip\_vs\_fo,ip\_vs\_nq,ip\_vs\_lblc,ip\_vs\_wrr,ip\_vs\_lc,ip\_vs\_sed,ip\_v\_ftp
nf\_defrag\_ipv6         20480  2 nf\_conntrack\_ipv6,ip\_vs
nf\_nat                 36864  3 nf\_nat\_ipv6,nf\_nat\_ipv4,ip\_vs\_ftp
nf\_conntrack          155648  9 xt\_conntrack,nf\_conntrack\_ipv6,nf\_conntrack\_ipv4,nf\_nat,nf\_nat\_ipv6,ipt\_MASQUERADE,nf\_nat\_ipv4,nf\_conntrack\_netlink,ip\_vs
libcrc32c              16384  4 nf\_conntrack,nf\_nat,xfs,ip\_vs

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有节点配置内核参数

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip\_forward \= 1
net.bridge.bridge\-nf-call-ip6tables = 1
net.bridge.bridge\-nf-call-iptables = 1
fs.may\_detach\_mounts \= 1
vm.overcommit\_memory\=1
vm.panic\_on\_oom\=0
fs.inotify.max\_user\_watches\=89100
fs.file\-max=52706963
fs.nr\_open\=52706963
net.netfilter.nf\_conntrack\_max\=2310720
EOF

sysctl \--system

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　配置Docker yum源（和1.16.x安装步骤一致）

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 k8s-ha-install\]# curl  https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2424  100  2424    0     0   3639      0 --:--:-- --:--:-- --:--:--  3645
\[root@k8s-master01 k8s-ha-install\]# yum makecache
CentOS-8 - AppStream                                                                                                                                                             559 kB/s | 6.3 MB     00:11    
CentOS-8 - Base                                                                                                                                                                  454 kB/s | 7.9 MB     00:17    
CentOS-8 - Extras                                                                                                                                                                592  B/s | 2.1 kB     00:03    
Docker CE Stable - x86\_64                                                                                                                                                        5.8 kB/s |  20 kB     00:03    
Last metadata expiration check: 0:00:01 ago on Sat 02 Nov 2019 02:46:29 PM CST.
Metadata cache created.

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有节点安装新版containerd

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 k8s-ha-install\]# wget https://download.docker.com/linux/centos/7/x86\_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86\_64.rpm
--2019-11-02 15:00:20--  https://download.docker.com/linux/centos/7/x86\_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86\_64.rpm
Resolving download.docker.com (download.docker.com)... 13.225.103.32, 13.225.103.65, 13.225.103.10, ...
Connecting to download.docker.com (download.docker.com)|13.225.103.32|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 27119348 (26M) \[binary/octet-stream\]
Saving to: ‘containerd.io-1.2.6-3.3.el7.x86\_64.rpm’

containerd.io-1.2.6-3.3.el7.x86\_64.rpm               100%\[===================================================================================================================\]  25.86M  1.55MB/s    in 30s     

2019-11-02 15:00:51 (887 KB/s) - ‘containerd.io-1.2.6-3.3.el7.x86\_64.rpm’ saved \[27119348/27119348\]

\[root@k8s-master01 k8s-ha-install\]# yum -y install containerd.io-1.2.6-3.3.el7.x86\_64.rpm
Last metadata expiration check: 0:14:35 ago on Sat 02 Nov 2019 02:46:29 PM CST.

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有节点安装最新版Docker

\[root@k8s-master01 k8s-ha-install\]# yum install docker-ce -y

　　所有节点开启Docker并设置开机自启动

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 k8s-ha-install\]# systemctl enable --now docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
\[root@k8s-master01 k8s-ha-install\]# docker version
Client: Docker Engine - Community
 Version:           19.03.4
 API version:       1.40
 Go version:        go1.12.10
 Git commit:        9013bf583a
 Built:             Fri Oct 18 15:52:22 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.4
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.10
  Git commit:       9013bf583a
  Built:            Fri Oct 18 15:50:54 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**4\. k8s组件安装**

　　下载kubernetes 1.17.x安装包

\[root@k8s-master01 ~\]# wget https://storage.googleapis.com/kubernetes-release/release/v1.17.0/kubernetes-server-linux-amd64.tar.gz

　　下载etcd 3.3.18安装包

\[root@k8s-master01 ~\]# wget https://github.com/etcd-io/etcd/releases/download/v3.3.18/etcd-v3.3.18-linux-amd64.tar.gz

　　解压kubernetes安装文件

\[root@k8s-master01 ~\]# tar -xf kubernetes-server-linux-amd64.tar.gz  --strip-components=3 -C /usr/local/bin kubernetes/server/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy}

　　解压etcd安装文件

\[root@k8s-master01 ~\]#  tar -zxvf etcd-v3.3.18-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-v3.3.18-linux-amd64/etcd{,ctl}

　　版本查看

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 ~\]# kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
\[root@k8s\-master01 ~\]# etcdctl -v
etcdctl version: 3.3.18
API version: 2

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　将组件发送到其他节点

MasterNodes='k8s-master02 k8s-master03'
WorkNodes='k8s-node01 k8s-node02'
for NODE in $MasterNodes; do echo $NODE; scp /usr/local/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy} $NODE:/usr/local/bin/; scp /usr/local/bin/etcd\* $NODE:/usr/local/bin/; done
for NODE in $WorkNodes; do     scp /usr/local/bin/kube{let,-proxy} $NODE:/usr/local/bin/ ; done

　　CNI安装，下载CNI组件

wget  https://github.com/containernetworking/plugins/releases/download/v0.7.5/cni-plugins-amd64-v0.7.5.tgz

　　所有节点创建/opt/cni/bin目录

mkdir -p /opt/cni/bin

　　解压cni并发送至其他节点

tar -zxf cni-plugins-amd64-v0.7.5.tgz -C /opt/cni/bin
for NODE in $MasterNodes; do     ssh $NODE 'mkdir -p /opt/cni/bin';     scp /opt/cni/bin/\* $NODE:/opt/cni/bin/; done
for NODE in $WorkNodes; do     ssh $NODE 'mkdir -p /opt/cni/bin';     scp /opt/cni/bin/\* $NODE:/opt/cni/bin/; done

**5\. 生成证书**

　　下载生成证书工具

wget "https://pkg.cfssl.org/R1.2/cfssl\_linux-amd64" -O /usr/local/bin/cfssl
wget "https://pkg.cfssl.org/R1.2/cfssljson\_linux-amd64" -O /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson

 　　所有Master节点创建etcd证书目录

mkdir /etc/etcd/ssl -p

 **k8s全栈架构师视频课程**

    51cto: [https://edu.51cto.com/sd/518e5](https://edu.51cto.com/sd/518e5)  
    腾讯课堂: [https://ke.qq.com/course/2738602](https://ke.qq.com/course/2738602)

　　购买前咨询QQ727585266领取大礼包

　　Master01节点生成etcd证书

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# pwd
/root/k8s-ha-install/pki

\[root@k8s-master01 pki\]#  cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca

\[root@k8s-master01 pki\]# cfssl gencert \\
   -ca=/etc/etcd/ssl/etcd-ca.pem \\
   -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \\
   -config=ca-config.json \\
   -hostname=127.0.0.1,k8s-master01,k8s-master02,k8s-master03,192.168.1.19,192.168.1.18,192.168.1.20 \\  
   -profile=kubernetes \\
   etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd

2019/12/26 22:48:00 \[INFO\] generate received request  
2019/12/26 22:48:00 \[INFO\] received CSR  
2019/12/26 22:48:00 \[INFO\] generating key: rsa-2048  
2019/12/26 22:48:01 \[INFO\] encoded CSR  
2019/12/26 22:48:01 \[INFO\] signed certificate with serial number 250230878926052708909595617022917808304837732033

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　将证书复制到其他节点

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# MasterNodes='k8s-master02 k8s-master03'
\[root@k8s-master01 pki\]# WorkNodes='k8s-node01 k8s-node02'
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# for NODE in $MasterNodes; do
     ssh $NODE "mkdir -p /etc/etcd/ssl"
     for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
       scp /etc/etcd/ssl/${FILE} $NODE:/etc/etcd/ssl/${FILE}
     done
 done
etcd-ca-key.pem                                                                                                                                                                100% 1675   424.0KB/s   00:00    

etcd-ca.pem                                                                                                                                                                    100% 1363   334.4KB/s   00:00    
etcd-key.pem                                                                                                                                                                   100% 1679   457.4KB/s   00:00    
etcd.pem                                                                                                                                                                       100% 1505   254.5KB/s   00:00    
etcd-ca-key.pem                                                                                                                                                                100% 1675   308.3KB/s   00:00    
etcd-ca.pem                                                                                                                                                                    100% 1363   479.0KB/s   00:00    
etcd-key.pem                                                                                                                                                                   100% 1679   208.1KB/s   00:00    
etcd.pem                                                                                                                                                                       100% 1505   398.1KB/s   00:00 

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　生成kubernetes证书

　　所有节点创建kubernetes相关目录

mkdir -p /etc/kubernetes/pki

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# cfssl gencert -initca ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca

\[root@k8s-master01 pki\]# cfssl gencert   -ca=/etc/kubernetes/pki/ca.pem   -ca-key=/etc/kubernetes/pki/ca-key.pem   -config=ca-config.json   -hostname=10.96.0.1,192.168.1.88,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,192.168.1.19,192.168.1.18,192.168.1.20   -profile=kubernetes   apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver

\[root@k8s-master01 pki\]# cfssl gencert   -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca 
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# cfssl gencert   -ca=/etc/kubernetes/pki/front-proxy-ca.pem   -ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem   -config=ca-config.json   -profile=kubernetes   front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client

\[root@k8s-master01 pki\]# cfssl gencert \\
   -ca=/etc/kubernetes/pki/ca.pem \\
   -ca-key=/etc/kubernetes/pki/ca-key.pem \\
   -config=ca-config.json \\
   -profile=kubernetes \\
   manager-csr.json | cfssljson -bare /etc/kubernetes/pki/controller-manager
\[root@k8s-master01 pki\]# kubectl config set-cluster kubernetes \\
     --certificate-authority=/etc/kubernetes/pki/ca.pem \\
     --embed-certs=true \\
     --server=https://192.168.1.88:8443 \\
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig


kubectl config set-context system:kube-controller-manager@kubernetes \\
    --cluster=kubernetes \\
    --user=system:kube-controller-manager \\
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

kubectl config use-context system:kube-controller-manager@kubernetes \\
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config set-credentials system:kube-controller-manager \\
     --client-certificate=/etc/kubernetes/pki/controller-manager.pem \\
     --client-key=/etc/kubernetes/pki/controller-manager-key.pem \\
     --embed-certs=true \\
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
User "system:kube-controller-manager" set.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config set-context system:kube-controller-manager@kubernetes \\
     --cluster=kubernetes \\
     --user=system:kube-controller-manager \\
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
Context "system:kube-controller-manager@kubernetes" created.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config use-context system:kube-controller-manager@kubernetes \\
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
Switched to context "system:kube-controller-manager@kubernetes".

\[root@k8s-master01 pki\]# cfssl gencert \\
   -ca=/etc/kubernetes/pki/ca.pem \\
   -ca-key=/etc/kubernetes/pki/ca-key.pem \\
   -config=ca-config.json \\
   -profile=kubernetes \\
   scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/scheduler

\[root@k8s-master01 pki\]# kubectl config set-cluster kubernetes \\
     --certificate-authority=/etc/kubernetes/pki/ca.pem \\
     --embed-certs=true \\
     --server=https://192.168.1.88:8443 \\
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
Cluster "kubernetes" set.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config set-credentials system:kube-scheduler \\
     --client-certificate=/etc/kubernetes/pki/scheduler.pem \\
     --client-key=/etc/kubernetes/pki/scheduler-key.pem \\
     --embed-certs=true \\
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
User "system:kube-scheduler" set.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config set-context system:kube-scheduler@kubernetes \\
     --cluster=kubernetes \\
     --user=system:kube-scheduler \\
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
Context "system:kube-scheduler@kubernetes" created.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config use-context system:kube-scheduler@kubernetes \\
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
Switched to context "system:kube-scheduler@kubernetes".

\[root@k8s-master01 pki\]# cfssl gencert \\
   -ca=/etc/kubernetes/pki/ca.pem \\
   -ca-key=/etc/kubernetes/pki/ca-key.pem \\
   -config=ca-config.json \\
   -profile=kubernetes \\
   admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin

\[root@k8s-master01 pki\]# kubectl config set-cluster kubernetes     --certificate-authority=/etc/kubernetes/pki/ca.pem     --embed-certs=true     --server=https://192.168.1.88:8443     --kubeconfig=/etc/kubernetes/admin.kubeconfig
Cluster "kubernetes" set.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config set-credentials kubernetes-admin     --client-certificate=/etc/kubernetes/pki/admin.pem     --client-key=/etc/kubernetes/pki/admin-key.pem     --embed-certs=true     --kubeconfig=/etc/kubernetes/admin.kubeconfig
User "kubernetes-admin" set.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config set-context kubernetes-admin@kubernetes     --cluster=kubernetes     --user=kubernetes-admin     --kubeconfig=/etc/kubernetes/admin.kubeconfig
Context "kubernetes-admin@kubernetes" created.
\[root@k8s-master01 pki\]# 
\[root@k8s-master01 pki\]# kubectl config use-context kubernetes-admin@kubernetes     --kubeconfig=/etc/kubernetes/admin.kubeconfig
Switched to context "kubernetes-admin@kubernetes".


\[root@k8s-master01 pki\]# for NODE in k8s-master01 k8s-master02 k8s-master03; do
     \\cp kubelet-csr.json kubelet-$NODE-csr.json;
     sed -i "s/\\$NODE/$NODE/g" kubelet-$NODE-csr.json;
     cfssl gencert \\
       -ca=/etc/kubernetes/pki/ca.pem \\
       -ca-key=/etc/kubernetes/pki/ca-key.pem \\
       -config=ca-config.json \\
       -hostname=$NODE \\
       -profile=kubernetes \\
       kubelet-$NODE-csr.json | cfssljson -bare /etc/kubernetes/pki/kubelet-$NODE;
     rm -f kubelet-$NODE-csr.json
   done

\[root@k8s-master01 pki\]# for NODE in k8s-master01 k8s-master02 k8s-master03; do
     ssh $NODE "mkdir -p /etc/kubernetes/pki"
     scp /etc/kubernetes/pki/ca.pem $NODE:/etc/kubernetes/pki/ca.pem
     scp /etc/kubernetes/pki/kubelet-$NODE-key.pem $NODE:/etc/kubernetes/pki/kubelet-key.pem
     scp /etc/kubernetes/pki/kubelet-$NODE.pem $NODE:/etc/kubernetes/pki/kubelet.pem
     rm -f /etc/kubernetes/pki/kubelet-$NODE-key.pem /etc/kubernetes/pki/kubelet-$NODE.pem
 done

# 如果Master节点需要部署kubelet，并且kubelet的kubeconfig也是用TLS BootStrap生成，此步骤可省略。
\[root@k8s-master01 pki\]# for NODE in k8s-master01 k8s-master02 k8s-master03; do
     ssh $NODE "cd /etc/kubernetes/pki && \\
       kubectl config set-cluster kubernetes \\
         --certificate-authority=/etc/kubernetes/pki/ca.pem \\
         --embed-certs=true \\
         --server=https://192.168.1.88:8443 \\
         --kubeconfig=/etc/kubernetes/kubelet.kubeconfig && \\
       kubectl config set-credentials system:node:${NODE} \\
         --client-certificate=/etc/kubernetes/pki/kubelet.pem \\
         --client-key=/etc/kubernetes/pki/kubelet-key.pem \\
         --embed-certs=true \\
         --kubeconfig=/etc/kubernetes/kubelet.kubeconfig && \\
       kubectl config set-context system:node:${NODE}@kubernetes \\
         --cluster=kubernetes \\
         --user=system:node:${NODE} \\
         --kubeconfig=/etc/kubernetes/kubelet.kubeconfig && \\
       kubectl config use-context system:node:${NODE}@kubernetes \\
         --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
 done

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 **k8s全栈架构师视频课程**

    51cto: [https://edu.51cto.com/sd/518e5](https://edu.51cto.com/sd/518e5)  
    腾讯课堂: [https://ke.qq.com/course/2738602](https://ke.qq.com/course/2738602)

　　购买前咨询QQ727585266领取大礼包

　　创建ServiceAccount Key

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]#  openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
...................................................................................+++++
...............+++++
e is 65537 (0x010001)
\[root@k8s-master01 pki\]# openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
writing RSA key  
  
\[root@k8s-master01 pki\]# 

for NODE in k8s-master02 k8s-master03; do   
for FILE in $(ls /etc/kubernetes/pki | grep -v etcd); do   
scp /etc/kubernetes/pki/${FILE} $NODE:/etc/kubernetes/pki/${FILE};  
done;   
for FILE in admin.kubeconfig controller-manager.kubeconfig scheduler.kubeconfig; do   
scp /etc/kubernetes/${FILE} $NODE:/etc/kubernetes/${FILE};  
done;  
done

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**6\. Kubernetes系统组件配置**

　　etcd配置大致相同，注意修改每个Master节点的etcd配置的主机名和IP地址

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\# cat _/etc/etcd/etcd.config.yml_

  
name: 'k8s-master01'
data-dir: /var/lib/etcd
wal-dir: /var/lib/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://192.168.1.19:2380'
listen-client-urls: 'https://192.168.1.19:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://192.168.1.19:2380'
advertise-client-urls: 'https://192.168.1.19:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'k8s-master01=https://192.168.1.19:2380,k8s-master02=https://192.168.1.18:2380,k8s-master03=https://192.168.1.20:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-output: default
force-new-cluster: false

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有Master节点创建etcd service并启动

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# cat /usr/lib/systemd/system/etcd.service
\[Unit\]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target

\[Service\]
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

\[Install\]
WantedBy=multi-user.target
Alias=etcd3.service

\[root@k8s-master01 pki\]# mkdir /etc/kubernetes/pki/etcd
\[root@k8s-master01 pki\]# ln -s /etc/etcd/ssl/\* /etc/kubernetes/pki/etcd/
\[root@k8s-master01 pki\]# systemctl daemon-reload
\[root@k8s-master01 pki\]# systemctl enable --now etcd
Created symlink /etc/systemd/system/etcd3.service → /usr/lib/systemd/system/etcd.service.
Created symlink /etc/systemd/system/multi-user.target.wants/etcd.service → /usr/lib/systemd/system/etcd.service.

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　高可用配置

　　所有Master节点安装keepalived和haproxy

yum install keepalived haproxy -y

　　HAProxy配置

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# cat /etc/haproxy/haproxy.cfg 
global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind \*:33305
  mode http
  option httplog
  monitor-uri /monitor

listen stats
  bind    \*:8006
  mode    http
  stats   enable
  stats   hide-version
  stats   uri       /stats
  stats   refresh   30s
  stats   realm     Haproxy\\ Statistics
  stats   auth      admin:admin

frontend k8s-master
  bind 0.0.0.0:8443
  bind 127.0.0.1:8443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default\_backend k8s-master

backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master01    192.168.1.19:6443  check
  server k8s-master02    192.168.1.18:6443  check
  server k8s-master03    192.168.1.20:6443  check

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 **k8s全栈架构师视频课程**

    51cto: [https://edu.51cto.com/sd/518e5](https://edu.51cto.com/sd/518e5)  
    腾讯课堂: [https://ke.qq.com/course/2738602](https://ke.qq.com/course/2738602)

　　购买前咨询QQ727585266领取大礼包

　　KeepAlived配置 \[root@k8s-master01 pki\]# vim /etc/keepalived/keepalived.conf ，注意每个节点的IP和网卡

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

! Configuration File for keepalived
global\_defs {
    router\_id LVS\_DEVEL
}
vrrp\_script chk\_apiserver {
    script "/etc/keepalived/check\_apiserver.sh"
    interval 2
    weight -5
    fall 3  
    rise 2
}
vrrp\_instance VI\_1 {
    state MASTER
    interface ens160
    mcast\_src\_ip 192.168.1.19
    virtual\_router\_id 51
    priority 100
    advert\_int 2
    authentication {
        auth\_type PASS
        auth\_pass K8SHA\_KA\_AUTH
    }
    virtual\_ipaddress {
        192.168.1.88
    }
    track\_script {  
      chk\_apiserver 

} }

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　健康检查配置

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 keepalived\]# cat /etc/keepalived/check\_apiserver.sh 
#!/bin/bash

err=0
for k in $(seq 1 5)
do
    check\_code=$(pgrep kube-apiserver)
    if \[\[ $check\_code == "" \]\]; then
        err=$(expr $err + 1)
        sleep 5
        continue
    else
        err=0
        break
    fi
done

if \[\[ $err != "0" \]\]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　启动HAProxy和KeepAlived

\[root@k8s-master01 keepalived\]# systemctl enable --now haproxy
\[root@k8s-master01 keepalived\]# systemctl enable --now keepalived

　　VIP测试

\[root@k8s-master01 pki\]# ping 192.168.1.88
PING 192.168.1.88 (192.168.1.88) 56(84) bytes of data.
64 bytes from 192.168.1.88: icmp\_seq=1 ttl=64 time=1.39 ms
64 bytes from 192.168.1.88: icmp\_seq=2 ttl=64 time=2.46 ms
64 bytes from 192.168.1.88: icmp\_seq=3 ttl=64 time=1.68 ms
64 bytes from 192.168.1.88: icmp\_seq=4 ttl=64 time=1.08 ms

　　Kubernetes组件配置

　　所有节点创建相关目录

\[root@k8s-master01 pki\]# mkdir -p /etc/kubernetes/manifests/ /etc/systemd/system/kubelet.service.d /var/lib/kubelet /var/log/kubernetes

　　所有Master节点创建kube-apiserver service

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# cat /usr/lib/systemd/system/kube-apiserver.service 
\[Unit\]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

\[Service\]
ExecStart=/usr/local/bin/kube-apiserver \\
      --v=2  \\
      --logtostderr=true  \\
      --allow-privileged=true  \\
      --bind-address=0.0.0.0  \\
      --secure-port=6443  \\
      --insecure-port=0  \\
      --advertise-address=192.168.1.88 \\
      --service-cluster-ip-range=10.96.0.0/12  \\
      --service-node-port-range=30000-32767  \\
      --etcd-servers=https://192.168.1.19:2379,https://192.168.1.18:2379,https://192.168.1.20:2379 \\
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem  \\
      --etcd-certfile=/etc/etcd/ssl/etcd.pem  \\
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem  \\
      --client-ca-file=/etc/kubernetes/pki/ca.pem  \\
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem  \\
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem  \\
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem  \\
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem  \\
      --service-account-key-file=/etc/kubernetes/pki/sa.pub  \\
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \\
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota  \\
      --authorization-mode=Node,RBAC  \\
      --enable-bootstrap-token-auth=true  \\
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem  \\
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem  \\
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem  \\
      --requestheader-allowed-names=aggregator  \\
      --requestheader-group-headers=X-Remote-Group  \\
      --requestheader-extra-headers-prefix=X-Remote-Extra-  \\
      --requestheader-username-headers=X-Remote-User  \\
      --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

\[Install\]
WantedBy=multi-user.target

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# vim /etc/kubernetes/token.csv 
\[root@k8s-master01 pki\]# cat !$
cat /etc/kubernetes/token.csv
d7d356746b508a1a478e49968fba7947,kubelet-bootstrap,10001,"system:kubelet-bootstrap"

　　所有Master节点开启kube-apiserver

\[root@k8s-master01 pki\]# systemctl daemon-reload && systemctl enable --now kube-apiserver

　　所有Master节点配置kube-controller-manager service

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# cat /usr/lib/systemd/system/kube-controller-manager.service
\[Unit\]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

\[Service\]
ExecStart=/usr/local/bin/kube-controller-manager \\
      --v=2 \\
      --logtostderr=true \\
      --address=127.0.0.1 \\
      --root-ca-file=/etc/kubernetes/pki/ca.pem \\
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \\
      --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \\
      --service-account-private-key-file=/etc/kubernetes/pki/sa.key \\
      --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \\
      --leader-elect=true \\
      --use-service-account-credentials=true \\
      --node-monitor-grace-period=40s \\
      --node-monitor-period=5s \\
      --pod-eviction-timeout=2m0s \\
      --controllers=\*,bootstrapsigner,tokencleaner \\
      --allocate-node-cidrs=true \\
      --cluster-cidr=10.244.0.0/16 \\
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \\
      --node-cidr-mask-size=24
      
Restart=always
RestartSec=10s

\[Install\]
WantedBy=multi-user.target

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有Master节点启动kube-controller-manager

\[root@k8s-master01 pki\]# systemctl daemon-reload

\[root@k8s-master01 pki\]# systemctl enable --now kube-controller-manager
Created symlink /etc/systemd/system/multi-user.target.wants/kube-controller-manager.service → /usr/lib/systemd/system/kube-controller-manager.service.

　　所有Master节点配置kube-scheduler service

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 pki\]# cat /usr/lib/systemd/system/kube-scheduler.service 
\[Unit\]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

\[Service\]
ExecStart=/usr/local/bin/kube-scheduler \\
      --v=2 \\
      --logtostderr=true \\
      --address=127.0.0.1 \\
      --leader-elect=true \\
      --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

\[Install\]
WantedBy=multi-user.target

\[root@k8s-master01 pki\]# systemctl daemon-reload

\[root@k8s-master01 pki\]# systemctl enable --now kube-scheduler
Created symlink /etc/systemd/system/multi-user.target.wants/kube-scheduler.service → /usr/lib/systemd/system/kube-scheduler.service.

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**7\.  TLS Bootstrapping配置**

　　在Master01创建bootstrap

kubectl config set-cluster kubernetes     --certificate-authority=/etc/kubernetes/pki/ca.pem     --embed-certs=true     --server=https://192.168.1.88:8443     --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
kubectl config set-credentials tls-bootstrap-token-user     --token=c8ad9c.2e4d610cf3e7426e --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
kubectl config set-context tls-bootstrap-token-user@kubernetes     --cluster=kubernetes     --user=tls-bootstrap-token-user     --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
kubectl config use-context tls-bootstrap-token-user@kubernetes     --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 bootstrap\]# cp /etc/kubernetes/admin.kubeconfig /root/.kube/config
\[root@k8s\-master01 bootstrap\]# kubectl create -f bootstrap.secret.yaml 
secret/bootstrap-token-c8ad9c created
clusterrolebinding.rbac.authorization.k8s.io/kubelet-bootstrap created
clusterrolebinding.rbac.authorization.k8s.io/node-autoapprove-bootstrap created
clusterrolebinding.rbac.authorization.k8s.io/node-autoapprove-certificate-rotation created
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**8\. Node节点配置**

　　复制证书至Node节点

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 bootstrap\]# for NODE in k8s-node01 k8s-node02; do
     ssh $NODE mkdir -p /etc/kubernetes/pki /etc/etcd/ssl /etc/etcd/ssl
     for FILE in etcd-ca.pem etcd.pem etcd-key.pem; do
       scp /etc/etcd/ssl/$FILE $NODE:/etc/etcd/ssl/
     done
     for FILE in pki/ca.pem pki/ca-key.pem pki/front-proxy-ca.pem bootstrap-kubelet.kubeconfig; do
       scp /etc/kubernetes/$FILE $NODE:/etc/kubernetes/${FILE}
 done
 done

etcd-ca.pem                                                                                                                                                                    100% 1363   314.0KB/s   00:00    
etcd.pem                                                                                                                                                                       100% 1505   429.1KB/s   00:00    
etcd-key.pem                                                                                                                                                                   100% 1679   361.9KB/s   00:00    
ca.pem                                                                                                                                                                         100% 1407   459.5KB/s   00:00    
ca-key.pem                                                                                                                                                                     100% 1679   475.2KB/s   00:00    
front-proxy-ca.pem                                                                                                                                                             100% 1143   214.5KB/s   00:00    
bootstrap-kubelet.kubeconfig                                                                                                                                                   100% 2291   695.1KB/s   00:00    
etcd-ca.pem                                                                                                                                                                    100% 1363   325.5KB/s   00:00    
etcd.pem                                                                                                                                                                       100% 1505   301.2KB/s   00:00    
etcd-key.pem                                                                                                                                                                   100% 1679   260.9KB/s   00:00    
ca.pem                                                                                                                                                                         100% 1407   420.8KB/s   00:00    
ca-key.pem                                                                                                                                                                     100% 1679   398.0KB/s   00:00    
front-proxy-ca.pem                                                                                                                                                             100% 1143   224.9KB/s   00:00    
bootstrap-kubelet.kubeconfig                                                                                                                                                   100% 2291   685.4KB/s   00:00

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有Node节点创建相关目录

mkdir -p /var/lib/kubelet /var/log/kubernetes /etc/systemd/system/kubelet.service.d /etc/kubernetes/manifests/

　　所有节点配置kubelet service（Master节点不部署Pod也可无需配置）

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 bootstrap\]# vim  /usr/lib/systemd/system/kubelet.service
\[root@k8s-master01 bootstrap\]# cat !$
cat /usr/lib/systemd/system/kubelet.service
\[Unit\]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

\[Service\]
ExecStart=/usr/local/bin/kubelet

Restart=always
StartLimitInterval=0
RestartSec=10

\[Install\]
WantedBy=multi-user.target

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 bootstrap\]# vim  /etc/systemd/system/kubelet.service.d/10-kubelet.conf
\[root@k8s-master01 bootstrap\]# cat !$
cat /etc/systemd/system/kubelet.service.d/10-kubelet.conf
\[Service\]
Environment="KUBELET\_KUBECONFIG\_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
Environment="KUBELET\_SYSTEM\_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET\_CONFIG\_ARGS=--config=/etc/kubernetes/kubelet-conf.yml"
Environment="KUBELET\_EXTRA\_ARGS=--node-labels=node.kubernetes.io/node='' --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google\_containers/pause-amd64:3.1"
ExecStart=
ExecStart=/usr/local/bin/kubelet $KUBELET\_KUBECONFIG\_ARGS $KUBELET\_CONFIG\_ARGS $KUBELET\_SYSTEM\_ARGS $KUBELET\_EXTRA\_ARGS

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 bootstrap\]# vim /etc/kubernetes/kubelet-conf.yml
\[root@k8s-master01 bootstrap\]# cat !$
cat /etc/kubernetes/kubelet-conf.yml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: cgroupfs
cgroupsPerQOS: true
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　启动所有节点kubelet

systemctl daemon-reload
systemctl enable --now kubelet

　　查看集群状态

![](https://img2018.cnblogs.com/blog/1095387/201912/1095387-20191227221846141-309454373.png)

　　Kube-Proxy配置

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

cd /root/k8s-ha-install  
kubectl -n kube-system create serviceaccount kube-proxy
kubectl create clusterrolebinding system:kube-proxy         --clusterrole system:node-proxier         --serviceaccount kube-system:kube-proxy
SECRET=$(kubectl -n kube-system get sa/kube-proxy \\
    --output=jsonpath='{.secrets\[0\].name}')
JWT\_TOKEN=$(kubectl -n kube-system get secret/$SECRET \\
--output=jsonpath='{.data.token}' | base64 -d)
PKI\_DIR=/etc/kubernetes/pki
K8S\_DIR=/etc/kubernetes
kubectl config set-cluster kubernetes     --certificate-authority=/etc/kubernetes/pki/ca.pem     --embed-certs=true     --server=https://192.168.1.88:8443     --kubeconfig=${K8S\_DIR}/kube-proxy.kubeconfig
kubectl config set-credentials kubernetes     --token=${JWT\_TOKEN}     --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
kubectl config set-context kubernetes     --cluster=kubernetes     --user=kubernetes     --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
kubectl config use-context kubernetes     --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

    51cto: [https://edu.51cto.com/sd/518e5](https://edu.51cto.com/sd/518e5)  
    腾讯课堂: [https://ke.qq.com/course/2738602](https://ke.qq.com/course/2738602)

　　购买前咨询QQ727585266领取大礼包

　　赋值Service文件

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 k8s-ha-install\]# for NODE in k8s-master01 k8s-master02 k8s-master03; do
     scp ${K8S\_DIR}/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
     scp kube-proxy/kube-proxy.conf $NODE:/etc/kubernetes/kube-proxy.conf
     scp kube-proxy/kube-proxy.service $NODE:/usr/lib/systemd/system/kube-proxy.service
 done
  
  
 
\[root@k8s-master01 k8s-ha-install\]# 
\[root@k8s-master01 k8s-ha-install\]# for NODE in k8s-node01 k8s-node02; do
     scp /etc/kubernetes/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
     scp kube-proxy/kube-proxy.conf $NODE:/etc/kubernetes/kube-proxy.conf
     scp kube-proxy/kube-proxy.service $NODE:/usr/lib/systemd/system/kube-proxy.service
 done

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　所有节点启动kube-proxy

\[root@k8s-master01 k8s-ha-install\]# systemctl daemon-reload
\[root@k8s-master01 k8s-ha-install\]# systemctl enable --now kube-proxy
Created symlink /etc/systemd/system/multi-user.target.wants/kube-proxy.service → /usr/lib/systemd/system/kube-proxy.service.

**9\. 安装calico**

　　安装Calico 3.11.1

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 Calico\]# cd /root/k8s-ha-install/Calico/

\[root@k8s-master01 Calico\]# kubectl create -f calico.yaml 
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 　　查看Calico状态

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 Calico\]# kubectl get po -n kube-system -owide
NAME                                       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
calico-kube-controllers-648f4868b8-pvz8k   1/1     Running   0          61s   10.244.85.193   k8s-node01     <none>           <none>
calico-node-54fdt                          1/1     Running   0          61s   192.168.1.20    k8s-master03   <none>           <none>
calico-node-7f9vj                          1/1     Running   0          61s   192.168.1.18    k8s-master02   <none>           <none>
calico-node-msctk                          1/1     Running   0          61s   192.168.1.19    k8s-master01   <none>           <none>
calico-node-rbxcr                          1/1     Running   0          61s   192.168.1.22    k8s-node02     <none>           <none>
calico-node-s5mb4                          1/1     Running   0          61s   192.168.1.21    k8s-node01     <none>           <none>
\[root@k8s-master01 Calico\]# kubectl get node
NAME           STATUS   ROLES    AGE   VERSION
k8s-master01   Ready    <none>   23h   v1.17.0
k8s-master02   Ready    <none>   23h   v1.17.0
k8s-master03   Ready    <none>   23h   v1.17.0
k8s-node01     Ready    <none>   23h   v1.17.0
k8s-node02     Ready    <none>   23h   v1.17.0
\[root@k8s-master01 Calico\]# kubectl cluster-info
Kubernetes master is running at https://192.168.1.88:8443

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**10、安装CoreDNS**

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 Calico\]# cd /root/k8s-ha-install/CoreDNS/
\[root@k8s-master01 CoreDNS\]# kubectl create -f coredns.yaml 
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　查看状态

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 CoreDNS\]# kubectl get po -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-648f4868b8-pvz8k   1/1     Running   0          2m25s
calico-node-54fdt                          1/1     Running   0          2m25s
calico-node-7f9vj                          1/1     Running   0          2m25s
calico-node-msctk                          1/1     Running   0          2m25s
calico-node-rbxcr                          1/1     Running   0          2m25s
calico-node-s5mb4                          1/1     Running   0          2m25s
coredns-76b74f549-7znbw                    1/1     Running   0          30s
\[root@k8s-master01 CoreDNS\]# kubectl logs -f coredns-76b74f549-7znbw -n kube-system
.:53
\[INFO\] plugin/reload: Running configuration MD5 = 8b19e11d5b2a72fb8e63383b064116a1
CoreDNS-1.6.6
linux/amd64, go1.13.5, 6a7a75e

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**11、 集群验证**

　　安装busybox

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

cat<<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　验证解析

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\[root@k8s-master01 CoreDNS\]# kubectl exec  busybox -n default -- nslookup kubernetes
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local

\[root@k8s-master01 CoreDNS\]# kubectl exec  busybox -n default -- nslookup kube-dns.kube-system
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kube-dns.kube-system
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

[![复制代码](//assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　至此集群安装完毕，其他组件可以参考本博客其他版本安装的文档。

　　参考《再也不踩坑的Kubernetes实战指南》

　　对K8S薄弱的，也可已参考视频学习：

    51cto: [https://edu.51cto.com/sd/518e5](https://edu.51cto.com/sd/518e5)  
    腾讯课堂: [https://ke.qq.com/course/2738602](https://ke.qq.com/course/2738602)

　　购买前咨询QQ727585266领取大礼包

赞助作者一杯奶茶:

![](https://img2018.cnblogs.com/blog/1095387/201911/1095387-20191103001629637-1866534736.png)

本文转自 <https://www.cnblogs.com/dukuan/p/12104600.html>，如有侵权，请联系删除。