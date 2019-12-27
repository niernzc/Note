

#### 修改hostname

```shell
 hostnamectl set-hostname k8s-master
```

`GA`阶段，即`General Available`阶段：比较稳定，基本功能都已实现，可以交给客户试用

## [使用Kubeadm安装k8s]( https://www.kubernetes.org.cn/5846.html )

* 安装docker/kuberlet

> 在所有结点上安装

```shell
[root@k8s-master ~]# curl -sSL https://kuboard.cn/install-script/v1.16.0/install-kubelet.sh 

[root@k8s-master ~]# vi install-kubelet.sh 
#!/bin/bash

# 在 master 节点和 worker 节点都要执行

# 安装 docker
# 参考文档如下
# https://docs.docker.com/install/linux/docker-ce/centos/ 
# https://docs.docker.com/install/linux/linux-postinstall/

# 卸载旧版本
yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine

# 设置 yum repository
yum install -y yum-utils \
device-mapper-persistent-data \

# 安装并启动 docker
systemctl start docker
yum install -y nfs-utils

# 关闭 防火墙

# 关闭 SeLinux
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭 swap
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
# 如果有配置，则修改
sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf
# 可能没有，追加
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
# 执行命令以应用
sysctl -p

# 配置K8S的yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 卸载旧版本
yum remove -y kubelet kubeadm kubectl

# 安装kubelet、kubeadm、kubectl
yum install -y kubelet-1.16.0 kubeadm-1.16.0 kubectl-1.16.0

# 修改docker Cgroup Driver为systemd
# # 将/usr/lib/systemd/system/docker.service文件中的这一行 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# # 修改为 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
# 如果不修改，在添加 worker 节点时可能会碰到如下错误
# [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". 
# Please follow the guide at https://kubernetes.io/docs/setup/cri/
sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service

# 设置 docker 镜像，提高 docker 镜像下载速度和稳定性
# 如果您访问 https://hub.docker.io 速度非常稳定，亦可以跳过这个步骤
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

# 重启 docker，并启动 kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet

docker version

echo -e "\033[31;1mKubernetes.org.cn发文审核周期长，请确保您正在使用 https://kuboard.cn/install/install-k8s.html 上的最新文档，并加入了在线答疑QQ群808894550，以避免碰到问题时无人解答
\033[0m"

```

* 初始化master结点

  ```shell
  [root@k8s-master ~]# export MASTER_IP=192.168.182.130
  [root@k8s-master ~]# export APISERVER_NAME=apiserver.demo
  [root@k8s-master ~]# export POS_SUBNET=10.100.0.1/20
  [root@k8s-master ~]# echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
  [root@k8s-master ~]# curl -sSL https://kuboard.cn/install-script/v1.16.0/init-master.sh | sh
  [init] Using Kubernetes version: v1.16.0
  [preflight] Running pre-flight checks
  error execution phase preflight: [preflight] Some fatal errors occurred:
  	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
  [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
  To see the stack trace of this error execute with --v=5 or higher
  cp: 无法获取"/etc/kubernetes/admin.conf" 的文件状态(stat): 没有那个文件或目录
  --2019-10-31 13:50:01--  https://docs.projectcalico.org/v3.8/manifests/calico.yaml
  正在解析主机 docs.projectcalico.org (docs.projectcalico.org)... 134.209.106.40, 2400:6180:0:d1::808:1
  正在连接 docs.projectcalico.org (docs.projectcalico.org)|134.209.106.40|:443... 已连接。
  已发出 HTTP 请求，正在等待回应... 200 OK
  长度：20796 (20K) [application/x-yaml]
  正在保存至: “calico.yaml”
  
  100%[===========================================================>] 20,796      4.26KB/s 用时 4.8s   
  
  2019-10-31 13:50:10 (4.26 KB/s) - 已保存 “calico.yaml” [20796/20796])
  
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  unable to recognize "calico.yaml": Get http://localhost:8080/api?timeout=32s: dial tcp [::1]:8080: connect: connection refused
  请确保您正在使用 https://kuboard.cn/install/install-k8s.html 上的最新K8S安装文档，并加入了在线答疑QQ群，以避免碰到问题时无人解答
  
  
  [root@k8s-master ~]# vim init-master.sh 
  
  #!/bin/bash
  
  # 只在 master 节点执行
  
  # 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
  rm -f ./kubeadm-config.yaml
  cat <<EOF > ./kubeadm-config.yaml
  apiVersion: kubeadm.k8s.io/v1beta2
  kind: ClusterConfiguration
  kubernetesVersion: v1.16.0
  imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
  controlPlaneEndpoint: "${APISERVER_NAME}:6443"
  networking:
    serviceSubnet: "10.96.0.0/16"
    podSubnet: "${POD_SUBNET}"
    dnsDomain: "cluster.local"
  EOF
  
  # kubeadm init
  # 根据您服务器网速的情况，您需要等候 3 - 10 分钟
  kubeadm init --config=kubeadm-config.yaml --upload-certs
  
  # 配置 kubectl
  rm -rf /root/.kube/
  mkdir /root/.kube/
  cp -i /etc/kubernetes/admin.conf /root/.kube/config
  
  # 安装 calico 网络插件
  # 参考文档 https://docs.projectcalico.org/v3.8/getting-started/kubernetes/
  rm -f calico.yaml
  wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml
  sed -i "s#192\.168\.0\.0/16#${POD_SUBNET}#" calico.yaml
  kubectl apply -f calico.yaml
  
  echo -e "\033[31;1m请确保您正在使用 https://kuboard.cn/install/install-k8s.html 上的最新K8S安装文档，
  并加入了在线答疑QQ群，以避免碰到问题时无人解答\033[0m"
  
  ```

  修改第65行的serviceSubnet=>问题依旧

## [官方安装文档(kubeadm)]( https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/ )

安装三个软件包各自的目的：

* `kubeadm`：用于初始化集群的指令
* `kubelet`：在集群中每个结点上用来启动pod和contaner等
* `kubectl`：用来与集群通信的命令行工具

```shell
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

# 将 SELinux 设置为 permissive 模式(将其禁用)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable kubelet && systemctl start kubelet
```

需要将`sysctl` 配置中 `net.bridge.bridge-nf-call-iptables` 被设为1 

# [官方安装文档（kubeadm->kubernetes）]( https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/ )

* 安装kubeadm
* 初始化`control-plane`结点
  `control-plane`结点是指` control plane components `所运行的机器，这些组件包括`etcd`(cluster database)，以及`API server`

然后我没看懂这篇文档就回到了之前的脚本：

```shell
[root@k8s-master ~]# ls
anaconda-ks.cfg  calico.yaml  init-master.sh  install-kubelet.sh  kubeadm-config.yaml
[root@k8s-master ~]# kubeadm init --config=kubeadm-config.yaml --upload-certs
···
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join apiserver.demo:6443 --token lfgf37.0dvrjjscmgb7dfaw \
    --discovery-token-ca-cert-hash sha256:d78988878b959c9a3e52e72171a2e5a5aec1f02752b2394b17803261f27fbdd5 \
    --control-plane --certificate-key e3423907bfa4edd8b41a0ab66ef8496c5861f6aa6cf57c911efdf49193b39e78

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use 
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join apiserver.demo:6443 --token lfgf37.0dvrjjscmgb7dfaw \
    --discovery-token-ca-cert-hash sha256:d78988878b959c9a3e52e72171a2e5a5aec1f02752b2394b17803261f27fbdd5 

```

太混乱了决定重启vm

