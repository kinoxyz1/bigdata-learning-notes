
* [一、环境准备](#%E4%B8%80%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
* [二、 安装docker](#%E4%BA%8C-%E5%AE%89%E8%A3%85docker)
* [三、安装 node 节点](#%E4%B8%89%E5%AE%89%E8%A3%85-node-%E8%8A%82%E7%82%B9)
* [四、获取 token](#%E5%9B%9B%E8%8E%B7%E5%8F%96-token)
* [五、查看是否安装成功](#%E4%BA%94%E6%9F%A5%E7%9C%8B%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F)


----
# 一、环境准备
CentOS:
```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.220.121 jz-desktop-01
192.168.220.122 jz-desktop-02
192.168.220.123 jz-desktop-03
EOF

# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```
Ubuntu:
```bash
# 关闭防火墙
ufw disable

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
# 临时
hostname <hostname>
# 永久
vim /etc/hostname
<hostname>

# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.220.121 jz-desktop-01
192.168.220.122 jz-desktop-02
192.168.220.123 jz-desktop-03
EOF

# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```

# 二、 安装docker
[CentOS7安装Docker](../docker/CentOS7安装Docker.md)

# 三、安装 node 节点
Ubuntu:

查看 k8s 版本:
```bash
$ apt-cache madison kubeadm
   kubeadm |  1.19.4-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.3-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   ...
```
安装
```bash
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update
# 安装指定版本
apt-get install -y kubelet-<version> kubeadm-<version> kubectl-<version>
# 例如: 1.19.4-00 版本
apt-get install -y kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4
# 安装最新版本
apt-get install -y kubelet kubeadm kubectl
```


CentOS:

查看 k8s 版本:
```bash
$ yum list kubelet --showduplicates | sort -r
已加载插件：fastestmirror
可安装的软件包
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
kubelet.x86_64                       1.9.9-0                          kubernetes
kubelet.x86_64                       1.9.8-0                          kubernetes
...
```
安装
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum clean all && yum makecache
yum list kubelet --showduplicates | sort -r
# 安装指定版本
yum install -y kubelet-<version> kubeadm-<version> kubectl-<version>
# 例如: 1.19.4-00 版本
yum install -y kubelet=1.19.4-00 kubeadm=1.19.4-00 kubectl=1.19.4-00
```


# 四、获取 token
在master 节点执行以下命令:
```bash
root@jz-desktop-01:~# kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
xe4dur.4s63knpjxin3pte2   22h         2020-11-17T11:25:03+08:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
```
token 过24小时会自动失效, 如果上步骤中没有token显示, 可创建一个token
```bash
$ kubectl token create
```
拿到 sha256 值:
```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'    # 来查看hash。
```

```bash
kubeadm join <master IP>:6443 --token <kubeadm token list 显示的值> --discovery-token-ca-cert-hash sha256:<上一步拿到的sha256值>
```

# 五、查看是否安装成功
在master节点
```bash
$ kubectl get nodes
root@jz-desktop-01:~# kubectl get nodes
NAME            STATUS     ROLES    AGE    VERSION
jz-desktop-01   NotReady   master   114d   v1.18.6
jz-desktop-02   Ready      <none>   114d   v1.18.6
jz-desktop-03   Ready      <none>   79m    v1.18.6
```