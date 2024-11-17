# Docker维护常用命令
```shell
# 查看容器(-a 查看所有状态的容器)
docker ps -a

# 进入容器
docker exec -it <容器id> /bin/bash(/sh)

# 使用当前容器执行命令
docker exec -it <容器id> <command>

#  将容器内文件复制到宿主机
docker cp [container Id]:/[sourcePath]/   /targetPath/  

# 查看容器日志 持续追踪日志，从最后100行日志开始
docker logs  -f --tail 100 <容器id>

# 查看容器详细部署信息(ip、集群网络等)
docker inspect <容器id>

# 停止容器运行 (若当前容器配置有状态管理，会拉起新的容器运行)
docker stop <容器id>

# 删除容器（仅停止运行状态的容器能够执行）
docker rm <容器id>

# 查看集群部署的所有服务
docker service ls

# 查看服务日志
docker service logs -f 

# 查看服务详细信息，如当前服务部署在那个服务器节点。
docker service ps <服务id>

# 查看服务部署任务栈运行状态
docker stack services <服务id>

# 登录远程仓库，从私库拉去镜像需要提前登录
docker login
```

# 清理Docker容器磁盘空间占用
* 一般情况容器需要清理的文件大都为日志文件。
* 清理文件df后可能存在磁盘占用未释放的情况。<a href='./服务器/Linux.md' title='清理文件占用磁盘权柄'>清理方法</a>
```shell
# 清理非运行状态下的容器、没有在使用的镜像文件
docker container prune -a

# 查看占用空间最多的镜像
docker images --format '{{.Size}}\t{{.Repository}}:{{.Tag}}\t{{.ID}}' | sort -h -r | column -t

# 命令可以进一步查看空间占用细节，以确定是哪个镜像、容器或本地卷占用过高空间
docker system df -v 

# 根据容器目录名称查找 容器 
docker ps -q | xargs docker inspect --format '{{.State.Pid}}, {{.Id}}, {{.Name}}, {{.GraphDriver.Data.WorkDir}}' | grep 
```




# 集群环境下服务管理
```shell

# 创建服务组（对docker-compose配置的所用容器进行服务创建，并统一由一个组名进行管理）
# docker会根据配置尝试对容器进行启动。
# 例docker-compose 服务名称为 jenkins 最终生成服务名为 test_jenkins
docker stack deploy -c docker-compose.yml <test(服务组名称)>

# 更新服务镜像（在第一次容器部署时，起到拉取镜像，确保容器启动) --image <镜像> <服务名称（服务组名称_容器名 例:test_jenkins）> 
docker service update --image ejiyuan/demo-docker:latest test_demo-docker

# 删除服务组(删除当前组内所有容器)
docker stack rm test_jenkins
```


# Doker集群管理

## Swarm 集群管理
```shell 
# 初始化集群
docker swarm init

# 查看加入集群，对应角色的token  worker|manager
docker swarm join-token worker

# 集群重新选举主节点
docker swarm init --force-new-cluster

# 加入集群
docker swarm join <join-token>

# 离开集群
docker swarm leave 
```

## node节点管理
```shell
# 查看集群节点
docker node ls

# 查看节点详细信息
docker node inspect <节点id>
docker inspect <节点id>

# 删除节点
docker node rm <节点id>
```

## Network集群网络管理
```shell
# 创建 overlay 集群网络
docker network create -d overlay --attachable wesoft-overlay(网络名称) --subnet 10.0.1.0/24(可自定义)  

```


# 清理docker镜像仓库
```shell 
# 查看镜像标签
curl -X GET http://<registry_host>:<registry_port>/v2/<image_name>/tags/list

# 查看镜像列表
curl -X GET http://<registry_host>:<registry_port>/v2/_catalog

# 查看 Registry 的信息：
curl -X GET http://<registry_host>:<registry_port>/v2/

# 分析镜像层数较多的镜像，仅显示占用前10的镜像名称：
registry garbage-collect --dry-run /etc/docker/registry/config.yml | awk -F : '{print \$1}' | sort | uniq -c | sort -rn -k1 | head -10 | grep -v "redis|jdk|php|mysql|nginx|apache|zk|elastic"


# 预执行Registry 清理操作 --dry-run
docker exec -it 7ca2a registry garbage-collect --dry-run /etc/docker/registry/config.yml

```

# Docker踩坑备注

## docker-compsoe.yml 文件配置
* 服务启动节点配置：
  swarm 集群部署服务启动docker-compose.yml文件无法指定具体ip:port；仅能够指定 node.role = manager/worker或node.hostname = nodeName
