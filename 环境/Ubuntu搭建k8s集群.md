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

## 安装 docker

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
