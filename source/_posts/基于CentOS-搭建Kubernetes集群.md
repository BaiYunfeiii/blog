---
title: 基于CentOS 搭建Kubernetes集群
date: 2019-05-21 21:51:38
tags:
    - Container
    - DevOps
---

## 安装Docker
Kubernetes 是基于Docker运行的，以此需要现在集群的每个节点上安装Docker。

1. 设置yum仓库

Install required packages. yum-utils provides the yum-config-manager utility, and device-mapper-persistent-data and lvm2 are required by the devicemapper storage driver.
```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

Use the following command to set up the stable repository.
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

2. 安装Docker CE
```
sudo yum install docker-ce docker-ce-cli containerd.io
```

3. 启动Docker服务
```
sudo systemctl start docker
```
将当前用户添加到docker用户组, 不然每次都要sudo
```
sudo usermod -aG docker $USER
```
断开ssh，重新连接，尝试不使用sudo执行docker命令，如：
```
docker ps
```

> 参考https://docs.docker.com/install/linux/docker-ce/centos/

### 使用systemd替代cgroups

> https://docs.docker.com/config/daemon/systemd/

## 安装kubenetes

需要安装以下三款工具

- kubeadm: the command to bootstrap the cluster.
- kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
- kubectl: the command line util to talk to your cluster.

### 添加Kubunetes的源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```

### Set SELinux in permissive mode (effectively disabling it) 
```
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### 安装kubelet kubeadm kubectl
```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### enable kubelet

```
systemctl enable --now kubelet
```

## 组建集群

### 配置Master节点
在mater节点执行, 初始化master节点
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
初始化成功后，可以看到以下内容
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.4.168:6443 --token eo1f0z.ebu45svkllv7qfjb \
    --discovery-token-ca-cert-hash sha256:2f19ee235910c790325f781afffafd0532507cc4c0e9fe22203a92dfdbb021e2
```

如提示中所说，复制一份配置文件到自己的目录下
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
安装Install Flannel networking:
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

复制一份join指令
```
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```

### 配置Node
执行刚刚复制的join命令
```
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```
注：如果join命令丢失，可以执行创建一个新的token，并把join命令打印出来
```
kubeadm token create --print-join-command
```

如果执行成功，可以回到Master节点，查看集群中的node
```
kubectl get nodes
```
如果配置成功的话，应该可以看到
```
NAME                      STATUS     ROLES    AGE   VERSION
yfbai2c.mylabserver.com   NotReady   <none>   10m   v1.14.2
yfbai3c.mylabserver.com   NotReady   master   13m   v1.14.2
```
此时，我的节点都是NotReady的，很奇怪。需要查看一下原因。
```
kubectl describe nodes yfbai2c.mylabserver.com
```
使用Fannel Networks之后就好了。 //TODO 看下原因
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
---------------------------------------
注：我在查看node列表的时候，出现了下面这个问题
```
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```
可能是k8s没有读取到正确的配置文件，我通过将环境变量`KUBECONFIG`配置为`$HOME/.kube/config`解决
```
export KUBECONFIG=$HOME/.kube/config
```