---
title: 'k8s-kubernetes'
---

k8s的安装部署以及的相关知识。

# 生产环境部署

## 部署前准备
部署环境：3台Centos7虚拟机

|节点名  |IP             |hostname |配置|
|------ |-------------- |---------|---|
|master |192.168.47.130 |master   |   | 
|node1  |192.168.47.128 |node1    |   | 
|node2  |192.168.47.131 |node2    |   | 

---
**TIP:**
可以开启SHELL工具的"发送键盘输入到所有会话"功能，同时操作3台虚拟机。如xshell：工具--发送键盘输入到所有会话，默认会打开所有会话的同步输入，对于不需要同步输入的终端点击上部的ON为OFF即可。

---

1. 升级kernel
>**NOTE:** Docker 对 Linux 内核版本的最低要求是3.10，如果内核版本低于 3.10 会缺少一些运行 Docker 容器的功能。这些比较旧的内核，在一定条件下会导致数据丢失和频繁恐慌错误。
```shell
# 先在 CentOS 7.× 启用 ELRepo 仓库:
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

#仓库启用后，你可以使用下面的命令列出可用的系统内核相关包:
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

# 接下来，安装最新的主线稳定内核:
yum --enablerepo=elrepo-kernel install kernel-ml

# 可选操作：设置 GRUB 默认的内核版本
sed -i 's/GRUB_DEFAULT=saved/GRUB_DEFAULT=0/g' /etc/default/grub
# 运行下面的命令来重新创建内核配置.
grub2-mkconfig -o /boot/grub2/grub.cfg

# 重启并查看内核版本
reboot
# 重启以后运行
uname -sr
# 可选操作：卸载旧内核
rpm -qa | grep kernel
```
2. 配置hosts
```shell        
# 在3台虚机上配置hosts，IP和节点名根据实际情况修改
echo -e "192.168.47.130 master\n192.168.47.128 node1\n192.168.47.131 node2" >> /etc/hosts
```
3. 修改hostname
```shell
# xxx根据节点替换成对应节点名
hostnamectl set-hostname xxx
```

## 安装部署
### 1. 安装Docker--容器运行时
```shell
# 卸载旧版本
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 安装yum-utils（提供yum-config-manager工具包,用于添加仓库等）
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安装最新版本 Docker Engine 和 containerd
sudo yum install docker-ce docker-ce-cli containerd.io

## 创建 /etc/docker 目录。
mkdir /etc/docker

# 设置 daemon。
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

```
**请注意：**
>  我们最后在 daemon.json 中设置了 `native.cgroupdriver=systemd` ，这是因为docker默认使用的 Cgroup 驱动是 cgroupfs ，这可以通过运行命令` docker info|grep cgroup `查看到。但Linux系统默认将systemd作为cgroup管理器，为每个进程分配 cgroup ，约束分配给进程的资源。 如果配置容器运行时（docker）和 kubelet 使用 cgroupfs，这意味着将有两个不同的 cgroup 管理器，当有两个管理器时，最终将对这些资源产生两种视图。并且某些节点配置 kubelet 和 docker 使用 cgroupfs，而节点上运行的其余进程则使用 systemd；这类节点在资源压力下会变得不稳定。
> 更改设置，令容器运行时和 kubelet 使用 systemd 作为 cgroup 驱动，以此使系统更为稳定。 请注意在 docker 下设置 native.cgroupdriver=systemd 选项。

**参考链接：**
- [Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/)
- [容器运行时](https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/)
- [docker商业版受限？请了解下cri-o](https://mp.weixin.qq.com/s?src=11&timestamp=1601261649&ver=2611&signature=1ofmOooIcm7UIMP2xwJoUbBhFnkrhyv6u7z9yf2bsgZ2hOaK21ZJpoVj81Eyb5fSoL9G1Qr2II6h9Rkyt95TTqrtwcVWWhWr7sI3T*ePta1cHdD9DrB0y1XZuOa8MsRs&new=1)

### 2. 安装kubelet kubeadm kubectl
- 方案一：[国内阿里云镜像](https://developer.aliyun.com/mirror/kubernetes?spm=a2c6h.13651102.0.0.3e221b11PcIqfr)

```shell
#ps: 由于官网未开放同步方式, 可能会有索引gpg检查失败的情况, 这时请用 yum install -y --nogpgcheck kubelet kubeadm kubectl 安装

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

```
        
 - 方案二：[Google镜像](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)需要能访问Google

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet

```
**请注意：**
- 通过运行命令 setenforce 0 和 sed ... 将 SELinux 设置为 permissive 模式可以有效的将其禁用。 这是允许容器访问主机文件系统所必须的，例如正常使用 pod 网络。 您必须这么做，直到 kubelet 做出升级支持 SELinux 为止。

- 一些 RHEL/CentOS 7 的用户曾经遇到过问题：由于` iptables `被绕过而导致流量无法正确路由的问题。您应该确保 在` sysctl `配置中的` net.bridge.bridge-nf-call-iptables `被设置为 1。设置方法如下：

  ```shell
  cat <<EOF >  /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF
  sysctl --system
  ```
- 确保在此步骤之前已加载了` br_netfilter `模块。这可以通过运行 ` lsmod | grep br_netfilter `来完成。要显示加载它，请调用` modprobe br_netfilter `

**参考链接：**
- [安装 kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [Kubernetes 镜像](https://developer.aliyun.com/mirror/kubernetes?spm=a2c6h.13651102.0.0.3e221b11PcIqfr)
- [Production Ready Kubernetes Cluster Deployment on Ubuntu](https://medium.com/@rakeshjain_17559/production-ready-kubernetes-cluster-deployment-on-ubuntu-8cccdf952774)

### 使用 kubeadm 创建集群

#### 1、安装前准备
1. 关闭swap
    ``` bash
    swapoff -a    临时关闭
    free          可以通过这个命令查看swap是否关闭了
    vim /etc/fstab  永久关闭 注释swap那一行
    ``` 
2. 





[学习 Kubernetes 基础知识](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/)

## 创建集群

集群图:

![image](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)


1.  [安装配置kubectl](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/)

2.  [安装Minukube](https://kubernetes.io/zh/docs/tasks/tools/install-minikube/)
3.  查看版本:

        minikube version

4.  启动:

参考链接:

- [安装Minikube](https://kubernetes.io/zh/docs/tasks/tools/install-minikube/)

## ![部署应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_02.svg?v=1469803628347){.align-bottom width="50px" height="30px"} 部署应用

## ![部署应用](https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_02.svg?v=1469803628347){.align-bottom width="50px" height="30px"} 部署应用
