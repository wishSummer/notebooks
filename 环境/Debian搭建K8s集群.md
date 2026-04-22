# Debian 搭建 K8s 集群

## 搭建环境准备

1. 修改hostname

 ```shell
 sudo hostnamectl set-hostname debian-node

 # 编辑hosts文件，添加集群内节点的IP和主机名映射,以及本地回环地址
 vim /etc/hosts

 127.0.0.1   localhost
 127.0.1.1   debian-node

 192.168.5.28   ubuntu-master
 192.168.5.29   debian-node
 ```

1. 关闭swap

    ```shell
    sudo swapoff -a
    # 永久关闭 swap，注释掉 /etc/fstab 中的 swap 行
    sudo sed -i '/ swap / s/^/#/' /etc/fstab
    ```

2. 加载内核模块

    ```shell
    sudo modprobe overlay
    sudo modprobe br_netfilter

    # 持久化内核模块加载
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    ```

3. 配置sysctl

    ```shell
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-iptables = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.ipv4.ip_forward = 1
   EOF

    # 应用sysctl参数
    sudo sysctl --system

    # 验证参数是否生效 sysctl net.ipv4.ip_forward 应 = 1
    ```

## 安装 containerd

1. 更新软件源

   ```shell
   sudo apt update
   sudo apt install -y curl gnupg apt-transport-https ca-certificates
   ```

2. 安装containerd

    ```shell
   sudo apt install -y containerd
    ```

3. 配置 containerd

    ```shell
   sudo mkdir -p /etc/containerd
   sudo containerd config default | sudo tee /etc/containerd/config.toml

    ```

4. 修改cgroup驱动为systemd

    ```shell
    sudo vim /etc/containerd/config.toml

    # 若使用 vim 编辑器，按下 i 进入编辑模式，找到 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options] 部分，将 SystemdCgroup 设置为 true：
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true

    ```
5. 重启 containerd 服务

    ```shell
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    ```

## 安装 K8s 组件 kubeadm、kubelet 和 kubectl

1. 添加镜像源

    ```shell
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
    > /etc/apt/sources.list.d/kubernetes.list
    ```

2. 安装依赖

    ```shell
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl gpg
    ```

3. 获取官方密钥

    ```shell
   # 创建 keyrings 目录
   sudo mkdir -p -m 755 /etc/apt/keyrings

   # 下载官方 GPG key
   sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
       | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    # 写入 sources.list
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
    udo tee /etc/apt/sources.list.d/kubernetes.list

    ```

4. 安装 kubeadm、kubelet 和 kubectl

    ```shell
   # 更新索引
   sudo apt update

   # 安装 kubelet kubeadm kubectl
   sudo apt install -y kubelet kubeadm kubectl

   # 锁定版本，防止自动升级
   sudo apt-mark hold kubelet kubeadm kubectl
    ```

5. 验证安装

    ```shell
    kubeadm version
    kubelet --version
    kubectl version --client
    ```

6.  子节点无法使用 kubectl 获取pod、node问题

    ```shell
    # 确保目录存在
    mkdir -p ~/.kube

    # 主节点 ~/.kube/config文件 copy到子节点
    scp user@master-node:~/.kube ~/.kube

    # 修正权限（如果是以 root 拷贝的，普通用户可能读不了）
    chown $(id -u):$(id -g) $HOME/.kube/config

    # 验证是否成功
    kubectl get nodes
    ```

7.  kube-proxy 启动报错
    1.  err="no support for primary IP family \"IPv4\""问题
        1.  现代内核中 iptables-nft 与 iptables-legacy 并存。如果系统默认指向 legacy 但内核模块或 CNI 插件（如 Calico）倾向于使用 nft，会导致接口调用失败。同时，残留的旧规则链会干扰新规则的建立。
        2.  解决方式： 拥抱 nftables，并彻底清理旧规则，同时补齐 IPVS 探测模块。

        ```shell
        # 1. 切换至现代 nft 模式
        sudo update-alternatives --set iptables /usr/sbin/iptables-nft
        sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-nft
        sudo update-alternatives --set ebtables /usr/sbin/ebtables-nft
 
        # 2. 彻底清空旧规则（防止旧链干扰，此步至关重要）
        sudo iptables -F && sudo iptables -X
        sudo iptables -t nat -F && sudo iptables -t nat -X

        # 3. 加载 IPVS 及相关模块（确保 kube-proxy 启动时的依赖探测全部通过）
        sudo modprobe ip_vs
        sudo modprobe ip_vs_rr
        sudo modprobe ip_vs_wrr
        sudo modprobe ip_vs_sh
        sudo modprobe nf_conntrack
        ```

8. coredns 启动报错
   1. Loop detected
      1. 原因： CoreDNS 默认继承宿主机的 /etc/resolv.conf。如果宿主机开启了 systemd-resolved（指向 127.0.0.53），CoreDNS 会将查询发给自己，导致无限循环。
      2. 解决方式： 
   ```shell
   # 编辑 /etc/resolv.conf
   # 调整 nameserver 映射地址

   domain localdomain
   search localdomain
   nameserver 192.168.130.2 # 当前网关地址
   ```