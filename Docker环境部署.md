# Docker
> docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
> 
- run         创建一个docker 镜像
- exec        在容器内运行命令
- ps          展示所有容器（运行状态）
- build       根据DockerFile创建镜像
- pull        拉取镜像
- push        推送镜像
- images      查看所有镜像
- login       登录远程仓库
- logout      从远程仓库退出登录
- search      搜索镜像资源
- version     Show the Docker version information
- info        docker详细信息以及状态

Management Commands:
- builder     Manage builds
- buildx*     Docker Buildx (Docker Inc., v0.12.1)
- compose*    Docker Compose (Docker Inc., v2.24.6)
- container   管理容器
- context     Manage contexts
- image       管理镜像
- manifest    Manage Docker image manifests and manifest lists
- network     管理网络
- plugin      管理插件
- system      管理docker
- trust       Manage trust on Docker images 
- volume      管理docker容器存储文件

Swarm Commands:
- config      Manage Swarm configs
- node        管理 Swarm集群节点
- secret      Manage Swarm secrets
- service     管理Swarm集群服务
- stack       管理Swarm任务栈
- swarm       管理Swarm

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Global Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and
                           default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket to connect to
  -l, --log-level string   Set the logging level ("debug", "info", "warn", "error", "fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit


# Docker 环境部署

## CentOs安装Docker

### 1.安装Docker-utils

- 卸载系统旧版本docker
```shell
yum remove docker  docker-common  docker-selinux   docker-engine

```
- 安装docker
``` shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 配置镜像仓库
``` shell
#官方镜像仓库地址
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#阿里镜像仓库地址
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 安装docker-ce 
```shell
yum install docker-ce

如需指定版本:
查看yum源支持的docker版本
yum list docker-ce --showduplicates |sort -r
指定版本命令 yum install docker-ce-20.10.22-3.el7
```
### 2.设置docker配置文件
- 指定docker默认存储路径，docker默认路径存储空间较小，为便于后期存放拉取的镜像和创建的容器，指定存储路径。


```shell
#进入目录文件：
/usr/lib/systemd/system/docker.service							#(docker 23.0.6版本修改daemon。json)
#ExecStart参数后添加 --graph配置
ExecStart=/usr/bin/dockerd --graph /home/docker					#参数为指定目录参数（23.0.6以前版本可以使用此方式）		


#若不存在则新建 /etc/docker/daemon.json文件,加入
#注册镜像仓库
#设置使用 json-file 驱动程序作为默认的日志驱动程序
#设置日志参数
#指定目录（23.0.6版本以后使用此方式指定）
{
"registry-mirrors": ["https://0exvcft2.mirror.aliyuncs.com"],  
"log-driver": "json-file",										
"log-opts": {"max-size":"500m", "max-file":"3"},				
"data-root": "/home/docker"									
}

#保存后重载配置文件
systemctl daemon-reload
#若防火墙已开启、docker已经在运行状态。重启docker
systemctl restart docker
```

### 3.关闭selinux

```shell
#查看selinux当前状态
getenforce

#修改selinux
#方式一：永久关闭，修改Linux配置文件/etc/selinux/config （需要重启服务器生效）
vim /etc/selinux/config
SELINUX=enforcing					#SELinux共有3个状态enforcing （执行中）、permissive （不执行但产生警告）、disabled（关闭）

#方式二：临时修改
setenforce 0 						#setenforce 0设置为permissive模式；setenforce 1 设置为enforcing模式；
```

### 4.启动docker
```shell
#启动docker前需启动firewalld防火墙

systemctl start firewalld		#启动firewalld防火墙

systemctl start docker			#启动docker
```



### 5.安装docker-compose

- 安装docker-compose

若无法安装找国内镜像站连接自行拼接

```shell
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

- 权限赋予
```shell
chmod +x /usr/local/bin/docker-compose
```

- 版本查看
```shell
docker-compose --version
```

### 6.docker集群创建

- 主节点创建集群
```shell
docker swarm init --advertise-addr 172.16.220.20(内网口IP地址)

如需指定端口: 
sudo docker swarm init --data-path-port 6789 --advertise-addr 172.16.220.20
```

- 创建overlay 网络（默认加入主节点集群）
```shell
docker network create -d overlay --attachable wesoft-overlay --subnet 10.0.1.0/24(可自定义)  
```

- 子节点加入集群
```shell
在主节点复制命令结果到子节点: docker swarm join-token worker
```

### 7.增加防火墙规则放行端口
```shell
TCP端口2377集群管理端口
TCP与UDP端口7946 节点之间通讯端口
TCP与UDP端口4789 overlay网络通讯端口

firewall-cmd --zone=public --add-port=2377/tcp --permanent
firewall-cmd --zone=public --add-port=7946/tcp --permanent
firewall-cmd --zone=public --add-port=7946/udp --permanent
firewall-cmd --zone=public --add-port=4789/tcp --permanent
firewall-cmd --zone=public --add-port=4789/udp --permanent

firewall-cmd --zone=public --add-port=6033/tcp --permanent
firewall-cmd --zone=public --add-port=8500/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --reload
```

### 8.服务器搭建应用服务
-  拉取现有服务器配置
```shell
#方式一：
scp	[srcouce]可多个文件名  [目标服务器用户名]@[文件放置目录]
scp -r /home/daygeek/2g/shell-script/ root@:/opt/backup/	#-r 复制目录
```

### 9.修改配置文件拉取路径

-  创建git分支
​分支名称需要与git2consul下的config.json以及所有镜像yml配置文件中的名称对应
```shell
#git2consul/config.json文件下的参数
"branches" : ["coral-ningbo-jinrong-config"],	#对应coral-config分支名称
```
-  修改所需应用docker-compose ip、配置文件映射参数

### 10.运行镜像文件

- 根据镜像按顺序进行环境搭建
```shell
#登录公司镜像仓库获取拉去镜像权限
docker login --username=hzwesoft --password=7ijnuinlkasd9uno%njdo hub.hzwesoft.com

#优先运行基础环境如网关、数据库、路由等镜像(consul、mysql、ridis、rabbitMQ、nginx等)
#无shell脚本的镜像
docker-compose up -d		#镜像目录下执行，运行docker-compose.yml配置
```

### 11.配置nginx

- 前端项目config目录下添加当前平台配置文件。（注意：修改文件内设计名称字段）
- 打包前端项目package.json配置文件添加对应build命令配置
- 前端项目打包后文件放入html目录下

### 12.卸载docker

- 卸载docker
```shell
yum remove docker-ce
rm -rf /var/lib/docker
rpm -qa | grep docker
yum remove 						#删除内容
```




## Ubuntu安装docker

### 卸载旧版本
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### 安装docker
1. 设置 Docker 的存储库
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc


# Add the repository to Apt sources:
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. 安装 Docker 包
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. 修改docker文件目录
```
新建 /etc/docker/daemon.json文件
{
 "registry-mirrors": ["https://0exvcft2.mirror.aliyuncs.com"],
 "log-driver": "json-file",
 "log-opts": {"max-size":"500m", "max-file":"3"},
 "data-root": "/home/docker"
}
```

4. 启动docker
```
安装firewalld: apt install firewalld

启动docker前需启动firewalld防火墙
启动firewalld防火墙: systemctl start firewalld
启动docker: systemctl start docker
```

5. 登录公司镜像仓库
```
docker login --username=hzwesoft --password=7ijnuinlkasd9uno%njdo hub.hzwesoft.com
```

### 以插件的形式安装docker compose
```
sudo apt-get install docker-compose-plugin
docker compose version

```
### docker集群创建
1. 主节点创建集群
```
docker swarm init --advertise-addr 172.16.220.20(内网口IP地址)

如需指定端口: 
sudo docker swarm init --data-path-port 6789 --advertise-addr 172.16.220.20
```

2. 创建overlay 网络(可自定义网段)
```
docker network create -d overlay --attachable wesoft-overlay --subnet 10.0.1.0/24 
```

3. 子节点加入集群
```
在主节点复制命令结果到子节点: docker swarm join-token worker
```

4. 增加防火墙规则
```
TCP端口2377集群管理端口
TCP与UDP端口7946 节点之间通讯端口
TCP与UDP端口4789 overlay网络通讯端口

firewall-cmd --zone=public --add-port=2377/tcp --permanent
firewall-cmd --zone=public --add-port=7946/tcp --permanent
firewall-cmd --zone=public --add-port=7946/udp --permanent
firewall-cmd --zone=public --add-port=6789/tcp --permanent
firewall-cmd --zone=public --add-port=6789/udp --permanent

firewall-cmd --zone=public --add-port=6033/tcp --permanent
firewall-cmd --zone=public --add-port=8500/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --zone=public --add-port=8883/tcp --permanent
firewall-cmd --zone=public --add-port=8884/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
```