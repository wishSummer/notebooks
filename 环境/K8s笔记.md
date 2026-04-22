# K8s 学习笔记

## K8s 基础概念

### namespace

- 主要用于对多套环境的资源隔离或多租户的资源隔离
- 可以限制namespace 内存、cpu等资源的使用，防止某个namespace 内的资源过度使用导致其他namespace 的资源无法使用
- default: 默认的 namespace，用户创建的资源如果没有指定 namespace 就会被放在 default namespace 中
- kube-system：Kubernetes 系统组件所在的 namespace，例如 kube-apiserver、kube-controller-manager、kube-scheduler 等组件都运行在 kube-system namespace 中
- kube-public：Kubernetes 公共资源所在的 namespace，通常用于存放一些公共的 ConfigMap 或 Secret 等资源，这些资源可以被集群中的所有用户访问

### node

### label

- label 是 Kubernetes 中用于标识和组织资源的键值对，可以附加在 Pod、Node、Service 等资源上。通过 label，可以对资源进行分组、选择和过滤，从而实现更灵活的资源管理和调度。
- labelSelector 是 Kubernetes 中用于选择具有特定 label 的资源的机制。通过 labelSelector，可以根据 label 的键值对来筛选和操作资源，例如在 Deployment 中使用 labelSelector 来选择要管理的 Pod。
  - matchLabels：通过指定一组键值对来选择具有这些 label 的资源，例如 matchLabels: { app: nginx } 会选择所有具有 app=nginx label 的资源。
  - matchExpressions：通过指定一组表达式来选择具有特定 label 的资源，例如 matchExpressions: [{ key: app, operator: In, values: [nginx, apache] }] 会选择所有具有 app=nginx 或 app=apache label 的资源。
  - matchFields：通过指定一组字段来选择具有特定字段值的资源，例如 matchFields: { metadata.name: nginx-pod } 会选择所有名称为 nginx-pod 的资源。
  - matchLabels 和 matchExpressions 可以同时使用，matchFields 只能单独使用。

### pod

- Pod 是 Kubernetes 中最小的可调度单元，表示集群中运行的一个或多个容器的集合。Pod 中的容器共享网络和存储资源，并且总是被调度到同一个节点上运行。
- pod 集群内分配到的IP为随机IP；并且仅集群内可见的虚拟IP地址，pod 之间通过这个IP地址进行通信；pod 之间的通信是通过 kube-proxy 组件实现的，kube-proxy 会在每个节点上运行一个代理进程，负责将 pod 的 IP 地址和端口映射到节点的 IP 地址和端口，从而实现 pod 之间的通信。
- pod 结构：
  ```shell

  # 一个 pod 中可以包含多个容器，这些容器共享网络和存储资源
  user contatiner

  user contatiner

  ......
  # 每个pod 都会有一个 Pause 容器，Pause 容器的作用是为 pod 提供一个共享的网络和存储环境，其他容器通过 Pause 容器来实现通信和数据共享；
  # Pause 容器本身不执行任何实际的工作，它只是一个占位符，用于保持 pod 的网络命名空间和存储卷的挂载状态；
  # 当 pod 中的其他容器需要访问网络或存储资源时，它们会通过 Pause 容器来实现通信和数据共享；
  # 如果 pod 中的所有容器都停止运行，Pause 容器也会停止运行，从而释放网络和存储资源。
  # 以 Pause 作为依据，评估整个 pod 的健康状态
  # 可以在根容器上设置 IP 地址，其他容器通过共享网络命名空间来访问这个 IP 地址，从而实现通信和数据共享
  Pause contatiner 
  ```
- pod 的生命周期：Pending -> Running -> Succeeded/Failed -> Unknown
  - 生命周期 状态转换
    - Pending：Pod 已经被 Kubernetes 接收，但尚未分配到节点上运行，可能是因为资源不足或调度器正在寻找合适的节点。
    - Running：Pod 已经被分配到节点上，并且至少有一个容器正在运行。
    - Succeeded：Pod 中的所有容器都已经成功完成并退出，通常用于一次性任务。
    - Failed：Pod 中的所有容器都已经完成，但至少有一个容器以非零状态退出，表示失败。
    - Unknown：Pod 的状态无法确定，可能是由于与节点通信失败或其他原因导致的。
  ```shell
  init container1
    init container2
      main container # 运行主容器过程 当前进程生命周期 包含下列进程
      post start  # 容器启动后执行的钩子，若执行成功则继续执行 main container 中的进程，否则重试 post start 钩子，直到成功为止
        liveness probe # 存活探针，定期检查容器是否存活，如果不存活则重启容器
        readiness probe # 就绪探针，定期检查容器是否就绪，如果不就绪则不接受流量
          pre stop # 容器停止前执行的钩子，若执行成功则继续停止容器，否则重试 pre stop 钩子，直到成功为止
  ```
  - pod 创建过程
    1. 用户通过 kubectl 或 其他api客户端提交需要创建的pod 信息到api-server
    2. Kubernetes API Server 接收到创建 Pod 的请求后，会将 Pod 对象存储在 etcd 中，然后返回确认信息至客户端
    3. api-server 开始反应etcd中pod 对象的变化，其他组件 使用watch机制监听 api-server 中 pod 的变化，并根据 pod 的状态进行相应的处理。
    4. scheduler 组件会监视 api-server 中的 pod 对象，并根据调度算法选择一个合适的节点来运行这个 pod，然后将调度结果更新到 api-server 中。
    5. Node 上的 kubelet 组件会监视 api-server 中的 pod 对象，并根据调度结果在节点上创建和管理 pod 中的容器，同时定期向 api-server 汇报 pod 的状态。
    6. api-server 会根据 kubelet 汇报的 pod 状态更新 pod 对象的状态，并通过 watch 机制通知其他组件和用户。

  - pod 终止过程
    1. 用户通过 kubectl 或其他 API 客户端提交删除 Pod 的请求到 API Server。
    2. api-server 中的 pod 对象 会在 宽限期内进行更新，宽限期是一个可配置的时间段，默认是 30 秒。在宽限期内，Pod 对象的状态会被更新为 Terminating。
    3. API Server 接收到删除 Pod 的请求后，会将 Pod 对象的状态更新为 Terminating，并将删除请求存储在 etcd 中。
    4. 端点控制器监控 API Server 中 Pod 对象的状态变化，当 Pod 进入 Terminating 状态时，端点控制器会将与该 Pod 相关的 Service 端点从负载均衡池中移除，以停止向该 Pod 发送流量。
    5. 如果当前 pod 定义了 preStop 钩子，kubelet 会在发送 SIGTERM 信号之前执行 preStop 钩子中定义的命令或 HTTP 请求。
    6. pod 对象中的容器在接收到 SIGTERM 信号后执行清理操作，例如关闭网络连接、保存状态等。
    7. 宽限期结束后，kubelet 会发送 SIGKILL 信号强制终止容器，并将 Pod 对象从 API Server 中删除。
    8. kubelet 请求 api-server 将 pod 对象 的宽限期设置为 0 ，以立即删除 pod 对象，此时pod 对用户不可见。

### deployment

- Deployment 是 Kubernetes 中的一种控制器资源，用于管理 Pod 的部署和更新。Deployment 可以确保指定数量的 Pod 副本在运行，并且可以自动进行滚动更新，以实现无停机的应用程序升级。
- Deployment 通过 labelSelector 来选择要管理的 Pod，Deployment 会监视这些 Pod 的状态，并根据需要创建、删除或更新 Pod，以确保满足用户指定的副本数量和更新策略。切记多个deployment 不能选择同一组pod，否则会发生冲突，导致无法正常工作。

### service

- 可以为一组pod提供一个稳定的访问入口，service 会为这些pod 分配一个虚拟IP地址，并且可以通过这个IP地址来访问这些pod；service 可以通过标签选择器来选择一组pod，这些pod 就会成为这个service 的后端；service 可以使用不同的类型来暴露服务，例如 ClusterIP、NodePort、LoadBalancer 等。

### ingress

### configmap

### secret

### volume

### statefulset

### daemonset

### job

### cronjob

### hpa

## k8s 组件

## k8s containerd 体系命令行区分

## K8s 集群搭建

## Ks 命令

### kubectl

- kubectl [command] [TYPE] [NAME] [flags]
  - command：指定要执行的操作，如 get、describe、logs 等
  - TYPE：资源类型，如 pod、service、deployment 等
  - NAME：资源名称，可以是具体的资源名称，也可以是标签选择器
  - flags：可选的命令行参数，用于指定额外的选项和行为

- 常用命令

    ```shell
    # 获取 所有 pods 信息
    kubectl get pods -A

    # 获取 指定 pods 信息，显示标签
    kubectl get pods <pod-name> --show-labels

    # 获取 指定 namespace 的 pods 信息
    kubectl get pods -n kube-systems

    # 查看 pod 详细信息
    kubectl describe pod <pod-name> -n <namespace>

    # 查看 pod 日志
    kubectl logs <pod-name> -n <namespace>

    # 查看某个pod，以yaml格式输出
    kubectl get pod <pod-name> -n <namespace> -o yaml

    # 查看某个pod，以wide格式输出，更详细的信息（包括节点信息）
    kubectl get pod <pod-name> -n <namespace> -o wide

    # 进入pod 内的 容器 内。
    kubectl exec nginx-7854ff8877-ff7f4 -n <namespace> -it -c nginx /bin/bash

    ```

- 命令分类

| 分类 | 命令 | 描述 |
|------|------|------|
| 基本命令 | get | 获取资源信息 |
| 基本命令 | create | 创建资源 |
| 基本命令 | delete | 删除资源 |
| 基本命令 | edit | 编辑资源 |
| 基本命令 | patch | 更新资源 |
| 运行调试命令 | run | 运行一个新的 pod |
| 运行调试命令 | expose | 暴露资源为服务 |
| 运行调试命令 | logs | 查看 pod 日志 |
| 运行调试命令 | describe | 查看资源详细信息 |
| 运行调试命令 | exec | 在 pod 中执行命令 |
| 运行调试命令 | attach | 连接到正在运行的 pod |
| 运行调试命令 | apply | 应用配置文件创建或更新资源 |
| 运行调试命令 | cp | 在 pod 和本地文件系统之间复制文件 |
| 运行调试命令 | rollout | 管理部署的滚动更新 -- todo 了解 |
| 运行调试命令 | scale | 调整资源的副本数量 |
| 运行调试命令 | autoscale | 自动调整资源的副本数量 |
| 高级命令 | apply | 应用配置文件创建或更新资源 |
| 高级命令 | label | 为资源添加或修改标签 |
| 高级命令 | annotate | 为资源添加或修改注释 |
| 其他命令 | cluster-info | 显示集群信息 |
| 其他命令 | version | 显示 kubectl 版本信息 |

- 资源分类

| 分类 | 资源名称 | 缩写 | 描述 |
|------|------|------|------|
| 集群级别资源 | nodes | no | 节点资源，表示集群中的计算资源 |
| 集群级别资源 | namespaces | ns | 命名空间资源，用于隔离和组织集群中的资源 |
| pod资源 | pods | po | Pod 资源，表示运行中的容器组 |
| pod资源控制器 | deployments | deploy | 部署资源，用于管理 Pod 的部署和更新 |
| pod资源控制器 | replicasets | rs | 副本集资源，用于确保指定数量的 Pod 副本在运行 |
| pod资源控制器 | replicationcontrollers | rc | 复制控制器资源，用于管理 Pod 的副本 |
| pod资源控制器 | statefulsets | sts | 有状态集合资源，用于管理有状态的 Pod |
| pod资源控制器 | daemonsets | ds | 守护进程集资源，确保每个节点上运行一个 Pod |
| pod资源控制器 | jobs | job | 任务资源，用于管理一次性任务 |
| pod资源控制器 | cronjobs | cj | 定时任务资源，用于管理周期性任务 |
| pod资源控制器 | horizontalpodautoscalers | hpa | 水平 Pod 自动扩缩资源，用于根据负载自动调整 Pod 副本数量 |
| 服务资源 | services | svc | 服务资源，用于暴露和负载均衡 Pod |
| 服务资源 | ingresses | ing | 入口资源，用于管理外部访问集群的 HTTP 和 HTTPS 路由 |
| 存储资源 | persistentvolumes | pv | 持久卷资源，用于提供持久存储 |
| 存储资源 | persistentvolumeclaims | pvc | 持久卷声明资源，用于请求持久存储 |
| 存储资源 | volumeattachments | va | 卷附件资源，用于管理卷与节点的关联 |
| 配置资源 | configmaps | cm | 配置映射资源，用于存储非机密的配置信息 |
| 配置资源 | secrets | secret | 密钥资源，用于存储敏感信息

- 创建资源的三种方式
  - 命令行参数方式：直接在命令行中指定资源的属性和配置，例如：

    ```shell
    kubectl run nginx --image=nginx:latest --replicas=3
    ```
  - 使用 kubectl apply 命令 + YAML 文件方式：通过编写 YAML 文件定义资源的属性和配置，然后创建资源，例如：

      ```shell
      # 可以更新或是创建资源，如果资源不存在则创建(create)，如果资源存在则更新(pacth)
      kubectl apply -f nginx-deployment.yaml
      ```
  
  - 使用 kubectl create 命令 + YAML 文件方式：通过编写 YAML 文件定义资源的属性和配置，然后创建资源，例如：

      ```shell
      # create 仅能够创建
      kubectl create -f nginx-deployment.yaml

      # patch 仅能够更新
      kubectl patch -f nginx-deployment.yaml
      ```
      
      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: nginx-deployment
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
            - name: nginx
              image: nginx:latest
              ports:
              - containerPort: 80
      ```

- label

  ```shell
  # 给 pod 添加 label
  kubectl label pod <pod-name> app=nginx

  # 给 deployment 添加 label
  kubectl label deployment <deployment-name> app=nginx

  # 给 node 添加 label
  kubectl label node <node-name> role=worker

  # 给 service 添加 label
  kubectl label service <service-name> app=nginx

  # 给 namespace 添加 label
  kubectl label namespace <namespace-name> env=production

  # 给 pod 添加多个 label
  kubectl label pod <pod-name> app=nginx tier=frontend

  # 删除 pod 上的 label, label key值后添加 - 表示删除
  kubectl label pod <pod-name> app-

  # 更新 pod 上的 label overwrite
  kubectl label pod <pod-name> app=apache --overwrite
  ```

- labelSelector
  
  ```shell
  # 获取具有特定 label 的 pod
  kubectl get pods -l app=nginx

  # 获取具有特定 label 的 deployment
  kubectl get deployments -l app=nginx
  kubectl get deployments -l app!=nginx

  # 模糊匹配
  kubectl get pods -l 'app in (nginx, apache)'

  ```

- depolyment
  
  ```shell
  # 创建 deployment
  kubectl create deployment nginx-deployment --image=nginx:latest

  # 创建 deployment，并指定副本数量 run 和 create deployment 的区别在于 run 可以直接指定副本数量，而 create deployment 只能创建一个副本，后续需要使用 scale 命令调整副本数量
  kubectl run nginx-deployment --image=nginx:latest --replicas=3 -n default

  # 更新 deployment 镜像版本
  kubectl set image deployment/nginx-deployment nginx=nginx:1.19.0

  # 扩缩 deployment 副本数量
  kubectl scale deployment/nginx-deployment --replicas=5

  # 删除 deployment
  kubectl delete deployment/nginx-deployment
  ```
- service

  ```shell
  # 创建 service, expose 命令会根据 deployment 的标签选择器来创建 service，并且会自动为 service 分配一个虚拟IP地址
  # 当前暴漏的端口依旧只能在集群内访问，如果需要暴露到集群外部，需要 --type 使用 NodePort 或 LoadBalancer 类型的 service
  kubectl expose deploy nginx-deployment --name=nginx-service --port=80 --target-port=80 --type=ClusterIP

  # 获取 service 信息
  kubectl get service/nginx-service

  # 删除 service
  kubectl delete service/nginx-service
  ```
### kubeadm

## yml 文件编写

- 使用 explain 命令来验证 YAML 文件的正确性，例如：

  ```shell
  # 查看 deployment 资源的字段和结构
  kubectl explain deployment

  # 查看 deployment 资源的 spec 字段的字段和结构
  kubectl explain deployment.spec

  # 查看 apiVersion 可填写版本
  kubectl api-versions

  # 查看 yml kind 可填写类型
  kubectl api-resources

  ```

### depolyment

### Pod

```yaml
apiVersion: v1 # 必选
kind: Pod # 必选 资源类型，当前为 Pod
metadata: # 必选 元素据
  name: nginx-pod # 必选 Pod 的名称
  namespace: default # 可选 Pod 所在的命名空间，默认为 default
  labels: # 可选 Pod 的标签，可以用于选择器和组织资源
    app: nginx # 可选 标签键值对

spec: # 必选 Pod 中容器的详细定义
  containers: # 必选 Pod 中的容器列表
  - name: nginx # 必选 容器的名称
    image: nginx:latest # 必选 容器使用的镜像
    imagePullPolicy: IfNotPresent # 可选 镜像拉取策略[Always|IfNotPresent|Never]，默认为 IfNotPresent
    command: [Sring] # 可选 容器启动时执行的命令，覆盖镜像中的默认命令
    args: [String] # 可选 容器启动时执行命令的参数，覆盖镜像中的默认参数
    workingDir: [string] # 可选 容器的工作目录，默认为根目录
    volumeMounts: # 可选 容器挂载的卷列表
    - name: config-volume # 可选 卷的名称，必须与 spec.volumes 中定义的卷名称匹配
      mountPath: /etc/config # 可选 卷在容器内的挂载路径
      readOnly: true # 可选 是否以只读方式挂载卷，默认为 false
    ports: # 可选 容器暴露的端口列表
    - name: [String] # 可选 端口的名称
      containerPort: [int] # 必选 容器需要监听的端口号，必须在 1-65535 之间
      hostPort: [int] # 可选 容器映射到主机的端口号
      hostIp: [String] # 可选 外部端口绑定到主机的 IP
      protocol: TCP # 可选 端口使用的协议[UDP|TCP]，默认为 TCP
    env: # 可选 容器的环境变量列表
    -name: [String] # 可选 环境变量的名称
     value: [String] # 可选 环境变量的值
     valueFrom: # 可选 环境变量的值来源，可以是 ConfigMap、Secret、FieldRef 等
       configMapKeyRef: # 可选 从 ConfigMap 中获取环境变量值
         name: [String] # 必选 ConfigMap 的名称
         key: [String] # 必选 ConfigMap 中的键，必须存在于指定的 ConfigMap 中
       secretKeyRef: # 可选 从 Secret 中获取环境变量值
         name: [String] # 必选 Secret 的名称
         key: [String] # 必选 Secret 中的键，必须存在于指定的 Secret 中
       fieldRef: # 可选 从 Pod 字段中获取环境变量值
         fieldPath: [String] # 必选 Pod 字段路径，例如 metadata.name、status.hostIP 等
    resources: # 可选 容器的资源请求和限制
      requests: # 可选 容器的资源请求，表示容器正常运行所需的最小资源量
        cpu: [String] # 可选 CPU 请求，例如 "100m" 表示 100 毫核
        memory: [String] # 可选 内存请求，例如 "128Mi" 表示 128 兆字节
      limits: # 可选 容器的资源限制，表示容器可以使用的最大资源量
        cpu: [String] # 可选 CPU 限制，例如 "500m" 表示 500 毫核
        memory: [String] # 可选 内存限制，例如 "512Mi" 表示 512 兆字节
    lifecycle: # 可选 容器的生命周期钩子
      postStart: # 可选 容器启动后执行的钩子
        exec: # 可选 执行命令的钩子
          command: [String] # 必选 要执行的命令和参数列表
      preStop: # 可选 容器停止前执行的钩子
        exec: # 可选 执行命令的钩子
          command: [String] # 必选 要执行的命令和参数列表
        HttpGet: # 可选 对 pod 内容器健康检查方法设置为HttpGet
          host: [String] # 可选 主机名
          port: [int] # 必选 端口号
          path: [String] # 必选 请求路径
          scheme: HTTP # 可选 请求使用的协议[HTTP|HTTPS]，默认为 HTTP
          Httpheaders: # 可选 请求头列表
          - name: [String] # 可选 请求头的名称
            value: [String] # 可选 请求头的值
        tcpSocket: # 可选 对 pod 内容器健康检查方法设置为TcpSocket
          port: [int] # 必选 端口号
      livenessProbe: # 可选 存活探针，定期检查容器是否存活，如果不存活则重启容器
        exec: # 可选 执行命令的探针
          command: [String] # 必选 要执行的命令和参数列表
        tcpSocket: # 可选 对 pod 内容器健康检查方法设置为TcpSocket
          port: [int] # 必选 端口号
        httpGet: # 可选 对 pod 内容器健康检查方法设置为HttpGet
          path: [String] # 必选 请求路径
          host: [String] # 可选 主机名
          port: [int] # 必选 端口号
          scheme: HTTP # 可选 请求使用的协议[HTTP|HTTPS]，默认为 HTTP
        initialDelaySeconds: [int] # 可选 容器启动后等待多长时间才开始执行生命周期钩子，默认为 0
        timeoutSeconds: [int] # 可选 生命周期钩子执行的超时时间
        priodSeconds: [int] # 可选 生命周期钩子执行的间隔时间
        successThreshold: [int] # 可选 连续成功多少次，认为是成功，默认为 1
        failureThreshold: [int] # 可选 连续失败多少次认为是失败，默认为 3，最小值 1
      readinessProbe: # 可选 就绪探针，定期检查容器是否就绪，如果不就绪则不接受流量
        exec: # 可选 执行命令的探针
          command: [String] # 必选 要执行的命令和参数列表
        initialDelaySeconds: [int] # 可选 容器启动后等待多长时间才开始执行生命周期钩子，默认为 0
        timeoutSeconds: [int] # 可选 生命周期钩子执行的超时时间，默认为 1 秒
        periodSeconds: [int] # 可选 生命周期钩子执行的间隔时间
        successThreshold: [int] # 可选 生命周期钩子成功的最小连续次数，默认为 1
        failureThreshold: [int] # 可选 生命周期钩子失败的最小连续次数，默认为 3
        securityContext: # 可选 容器的安全上下文
          privileged: [bool] # 可选 是否以特权模式运行容器，默认为 false
  restartPolicy: Always # 可选 Pod 的重启策略[Always|OnFailure|Never]，默认为 Always
  nodeName: [String] # 可选 指定 Pod 运行的节点名称，默认为空表示由 Kubernetes 调度器自动选择节点
  nodeSelector: [object] # 可选 指定 Pod 运行的节点标签选择器，只有具有匹配标签的节点才会被调度器考虑
    disktype: ssd # 可选 节点标签键值对
  imagePullSecrets: # 可选 Pod 使用的镜像拉取密钥列表，用于从私有镜像仓库拉取镜像
  - name: [String] # 可选 镜像拉取密钥的名称
  hostNetwork: [bool] # 可选 是否使用主机网络，默认为 false
  volumes: # 可选 定义 pod 与宿主机共享存储列表
  - name: [String] # 可选 卷的名称，必须与容器中的 volumeMounts.name 匹配
    hostPath: [String] # 可选 使用宿主机路径的卷，path 指定宿主机上的路径，type 可选目录、文件、套接字等
      path: [String] # 必选 宿主机上的路径
    emptyDir: [String] # 可选 使用空目录的卷，pod 删除后数据会被删除
    secret: # 类型为 Secret 的卷，使用 Kubernetes Secret 存储敏感数据
      secretName: [String] # 必选 Secret 的名称
      items: # 可选 从 Secret 中选择特定键作为卷中的文件
      - key: [String] # 必选 Secret 中的键
        path: [String] # 必选 卷中对应的文件路径
        
```