# Ubuntu 搭建k8s集群

## Ubuntu 安装 `ssh`

1. 确认是否安装了 `ssh` 服务

   ```shell
    sudo systemctl status ssh
   ```

2. 安装 `ssh`

   ```shell
   sudo apt update
   sudo apt install openssh-server
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```

3. 检查防火墙 `ufw` 是否开启

    ```shell
    sudo ufw status
    ```

4. 若 `ufw` 启用，配置允许 ssh 端口

    ```shell
    sudo ufw allow ssh
    sudo ufw reload
    ```

## Ubuntu 磁盘挂载

1. 确认是否存在闲置磁盘

   ```shell
   lsblk -f

   #输出结果如下：
    NAME      FSTYPE FSVER LABEL                           UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
    sda
    ├─sda1
    ├─sda2    ext4   1.0                                 8438212c-cbb8-4b48-b0aa-d288899bf1c5      1.7G     5% /boot
    └─sda3    LVM2_m LVM2                                g1zqFR-SCWV-bApE-hHw7-MoPu-WnP0-WIYtAV
    ├─ubuntu--vg-ubuntu--lv
    │       ext4   1.0                                   cff4807e-07c0-4921-bc17-70afc0a94265       16G    27% /
    └─ubuntu--vg-homelv
            ext4   1.0                                   f4edb5b3-a976-4002-b797-7de8ec4e52a1     22.2G     0% /home
    sr0       iso966 Jolie Ubuntu-Server 24.04.2 LTS amd64 2025-02-16-22-49-22-00
   ```

2. 创建磁盘分区

   ```shell
    # 创建逻辑卷（将 100% 的空闲空间用掉）
    sudo lvcreate -l 100%FREE -n homelv ubuntu-vg

    # 格式化为 ext4 文件系统
    sudo mkfs.ext4 /dev/ubuntu-vg/homelv

    # 临时挂载用于迁移旧数据（如果你已备份可以跳过）
    sudo mkdir /mnt/newhome
    sudo mount /dev/ubuntu-vg/homelv /mnt/newhome

    # 将原有数据迁移到新分区（如果有）
    sudo cp -a /home.bak/. /mnt/newhome/

    # 卸载旧 home，新 home 挂载   若无需备份直接 mount 剩余磁盘到目标目录
    sudo umount /mnt/newhome
    sudo mount /dev/ubuntu-vg/homelv /home

    # 修改 /etc/fstab
    sudo nano /etc/fstab

    # 添加如下内容
    /dev/ubuntu-vg/homelv /home ext4 defaults 0 2
    ```

## 安装 K8s 前置动作

1. 关闭swap

   ```shell
   sudo swapoff -a
   # 永久关闭 swap，注释掉 /etc/fstab 中的 swap 行
   sudo sed -i '/swap/d' /etc/fstab
   ```

2. 设置主机名
   ```shell
   sudo hostnamectl set-hostname master-node
   ```
3. 编辑hosts文件，添加集群内节点的IP和主机名映射

   ```shell
   sudo nano /etc/hosts

   # 添加 当前主机IP 与 主机名映射
   192.168.1.100   master-node
   ```

4. 同步时间

   ```shell
   sudo apt install chrony -y
   sudo systemctl enable chrony
   sudo systemctl start chrony
   chronyc tracking
   ```

5. 加载内核模块
   ```shell
   sudo modprobe overlay
   sudo modprobe br_netfilter

   # 确保模块在重启后自动加载
   sudo tee /etc/modules-load.d/k8s.conf <<EOF
   overlay
   br_netfilter
   EOF
   ```
6. 添加网络转发参数

   ```shell
   sudo tee /etc/sysctl.d/k8s.conf <<EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.ipv4.ip_forward = 1
   EOF

   # 应用 sysctl 参数
   sudo sysctl --system

   # 验证参数是否生效
   sysctl net.bridge.bridge-nf-call-iptables
   sysctl net.ipv4.ip_forward
   ```

7. 文件描述符优化

   ```shell
   sudo tee -a /etc/security/limits.conf <<EOF
   * soft nofile 1048576
   * hard nofile 1048576
   * soft nproc  1048576
   * hard nproc  1048576
   EOF
   ```

8. TCP 优化（高并发）

   ```shell
   sudo tee /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
   net.core.somaxconn = 32768
   net.core.netdev_max_backlog = 16384
   net.ipv4.tcp_max_syn_backlog = 8192
   net.ipv4.tcp_tw_reuse = 1
   EOF

   sudo sysctl --system
   ```

## 使用 docker 或 Containered

### 安装 docker

1. 查看是否安装有旧版本 docker

   ```shell
   sudo docker -v

   # 查看已安装软件列表
   apt list --installed

   # 卸载旧版本 docker
   sudo apt-get remove docker docker-engine docker.io containerd runc
   ```

2. 安装必要工具和依赖

   ```shell
   # 若 `ubuntu` 版本为 `noble` ，可能存在 `docker` 官方还为给 `noble` 发布包。需要添加 `jammy` 版本的 Docker 仓库（替代 noble）
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu jammy stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   sudo apt-get update
   sudo apt-get install -y ca-certificates curl gnupg lsb-release
   ```

3. 添加 docker 官方 GPG 密钥

   ```shell
   echo \
   "deb [arch=$(dpkg --print-architecture) \
   signed-by=/etc/apt/keyrings/docker.gpg] \
   https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable" | \
   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

4. 安装 docker

   ```shell
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   ```

5. 将当前用户加入 docker 用户组，避免每次都 sudo

   ```shell
   # 将当前用户加入 docker 用户组
   sudo usermod -aG docker $USER

   # 刷新用户组
   newgrp docker
   ```

6. 添加 docker 镜像加速

   ```shell
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'

    {
    "registry-mirrors": [
            "https://registry.cn-hangzhou.aliyuncs.com",
            "https://mirror.ccs.tencentyun.com",
            "https://registry.docker-cn.com",
            "http://hub-mirror.c.163.com",
            "https://docker.mirrors.ustc.edu.cn",
            "https://dockerhub.azk8s.cn",
            "https://mirror.baidubce.com/",
            "https://docker.melikeme.cn",
            "https://pull.loridocker.com",
            "https://docker.kejilion.pro",
            "https://docker.imgdb.de"
    ],
    "log-driver": "json-file",
    "log-opts": {"max-size":"500m", "max-file":"1"},
    "exec-opts": ["native.cgroupdriver=systemd"],
    "data-root": "/home/docker"
    }
    EOF

    sudo systemctl daemon-reexec
    sudo systemctl restart docker
   ```

### 安装 Contained

1. 安装 containerd

   ```shell
   sudo apt update
   sudo apt install -y containerd
   ```

2. 生成默认配置 containerd

   ```shell
   sudo mkdir -p /etc/containerd
   sudo containerd config default | sudo tee /etc/containerd/config.toml
   ```

3. 修改 containerd 配置

   ```shell
   # 修改 cgroup 驱动为 systemd
   SystemdCgroup = true

   # 重启 containerd 服务  
   sudo systemctl restart containerd
   sudo systemctl enable containerd

   # 验证 containerd 是否正常运行
   sudo systemctl status containerd
   ```

## 安装 K8s 组件 kubeadm、kubelet 和 kubectl

1. 添加下载源

   ```shell

   ```

2. 安装 kubeadm、kubelet 和 kubectl

   ```shell
   sudo apt update
   sudo apt install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

3. 验证安装

   ```shell
   kubeadm version
   kubelet --version
   kubectl version --client
   ```

4. 配置 kubelet 使用 systemd 作为 cgroup 驱动 ， kubelet_poxy 默认使用 ipvs

   ```shell
   sudo tee /etc/default/kubelet <<-'EOF'
   KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
   KUBE_PROXY_MODE=ipvs
   EOF

   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

5. 配置开机自启动

   ```shell
   sudo systemctl enable kubelet
   ```

## 配置镜像源配置

1. 编辑 containerd 配置文件,修改 config_path

      ```shell
      sudo vi /etc/containerd/config.toml


      # 在 [plugins."io.containerd.grpc.v1.cri".registry] 部分添加如下内容：

      # 找到这一块，把空的改掉
      [plugins."io.containerd.cri.v1.images".registry]
        config_path = "/etc/containerd/certs.d"

      # 同时也保留（或确保）这一块
      [plugins."io.containerd.grpc.v1.cri".registry]
        config_path = "/etc/containerd/certs.d"
      ```

2. 为 Docker Hub (docker.io) 配置加速
      ```shell
      # 创建目录和配置文件
      sudo mkdir -p /etc/containerd/certs.d/docker.io
      sudo vi /etc/containerd/certs.d/docker.io/hosts.toml

      # 写入以下内容

      server = "https://registry-1.docker.io"

      [host."https://docker.m.daocloud.io"]
        capabilities = ["pull", "resolve"]

      [host."https://hub-mirror.c.163.com"]
        capabilities = ["pull", "resolve"]

      [host."https://mirror.ccs.tencentyun.com"]
        capabilities = ["pull", "resolve"]
3. 创建镜像源配置文件,为 Google 镜像 (registry.k8s.io) 配置加速

      ```shell
      # 创建目录和配置文件
      sudo mkdir -p /etc/containerd/certs.d/registry.k8s.io
      sudo vi /etc/containerd/certs.d/registry.k8s.io/hosts.toml

      # 写入以下内容

      server = "https://registry.k8s.io"

      [host."https://k8s.m.daocloud.io"]
        capabilities = ["pull", "resolve"]
      ```

4. 重启服务使基础路径生效

      ```shell
      sudo systemctl restart containerd
      ```



## 初始化K8s集群

1. 查看init所需镜像列表

   ```shell
   # 可指定版本
   kubeadm config images list --kubernetes-version=v1.28.2
   ```


2. 手动下载init所需的镜像

   ```shell
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.28.2
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.28.2
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.28.2
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/kube-proxy:v1.28.2
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/pause:3.9
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/etcd:3.5.9-0
   sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/coredns:v1.10.1
   ```

3. 将下载的镜像重新打标签为 k8s.gcr.io 的官方镜像标签

   ```shell
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.29.15 registry.k8s.io/kube-apiserver:v1.29.15
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.29.15 registry.k8s.io/kube-controller-manager:v1.29.15
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.29.15 registry.k8s.io/kube-scheduler:v1.29.15
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/kube-proxy:v1.29.15 registry.k8s.io/kube-proxy:v1.29.15
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/pause:3.9 registry.k8s.io/pause:3.9
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/etcd:3.5.16-0 registry.k8s.io/etcd:3.5.16-0
   sudo ctr -n k8s.io images tag registry.aliyuncs.com/google_containers/coredns:v1.11.1 registry.k8s.io/coredns/coredns:v1.11.1
   ```

4. 验证镜像是否存在

   ```shell
   sudo crictl images
   ```

5. 初始化 master 节点

   ```shell
   sudo kubeadm init \
   --apiserver-advertise-address=192.168.130.132 \
   --pod-network-cidr=10.244.0.0/16 \
   --service-cidr=10.96.0.0/12 \
   --control-plane-endpoint=192.168.130.132


   # 初始化成功会显示 kubeadm join 命令，记录下来后续 worker 节点加入集群使用 
    kubeadm join 192.168.130.132:6443 --token 6kd7aj.o0k5qhpnn8zhhqnl \
        --discovery-token-ca-cert-hash sha256:44ec6a7a4a9ec081185e99cddd74d825a5fa2fbfed44fa5de6e6e4f2f91e5cfd \
        --control-plane
        kubeadm join 192.168.130.132:6443 --token scngjl.i85sdluww804wr4o \
        --discovery-token-ca-cert-hash sha256:0cf81d95b4f3e9714c8d48cd06fea18fac5552be4d62ee07bd3e2f5539bff923

   # 创建新的 token
   kubeadm token create 
   ```

   ```shell
   # 若初始化失败，再次初始化需要清理上次初始化的残留文件

   # reset 集群
   sudo kubeadm reset -f

   # 手动删除残留目录
   sudo rm -rf /etc/kubernetes/
   sudo rm -rf /var/lib/etcd
   sudo rm -rf /var/lib/kubelet/*

   # 重启 containerd 和 kubelet
   sudo systemctl restart containerd
   sudo systemctl restart kubelet

   # 确认端口释放
   sudo ss -lntp | grep 10250

   ```


6. 配置 kubectl 访问集群

   ```shell
   # 仅配置当前用户的 kubeconfig，其他用户需要重复执行以下命令
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

7. 创建子节点join-token

   ```shell
   # 创建 join-token，默认有效期 24 小时
   kubeadm token create --print-join-command
   ```

## 安装网络插件

1. 以安装 calico 网络插件为例

   ```shell
   kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
   ```

2. 验证网络插件是否正常运行

   ```shell
   kubectl get pods
   ```

3. 镜像下载问题
   
   ```shell
   # 配置containerd镜像加速
   sudo mkdir -p /etc/containerd/certs.d/docker.io
   sudo vim /etc/containerd/certs.d/docker.io/hosts.toml
   # 添加如下内容
   [host."https://docker.m.daocloud.io"]
   capabilities = ["pull", "resolve"]

   # 查看calico所需的镜像列表
   kubectl get ds -n kube-system calico-node -o yaml | grep image

   # 查看已下载的镜像列表
   sudo crictl images |grep calico

   # 手动调整 image 标签
   sudo ctr -n k8s.io images tag docker.m.daocloud.io/calico/node:v3.27.3 docker.io/calico/node:v3.25.0
   sudo ctr -n k8s.io images tag docker.m.daocloud.io/calico/cni:v3.27.3 docker.io/calico/cni:v3.25.0
   sudo ctr -n k8s.io images tag docker.m.daocloud.io/calico/kube-controllers:v3.27.3 docker.io/calico/kube-controllers:v3.25.0
   sudo ctr -n k8s.io images tag docker.m.daocloud.io/calico/pod2daemon-flexvol:v3.27.3 docker.io/calico/pod2daemon-flexvol:v3.25.0

   ```
