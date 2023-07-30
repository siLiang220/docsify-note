## 1.K8s功能和架构

### 1.1 k8s功能
- 自动装箱
	- 基于容器对应用运行环境的资源配置要求自动部署应用容器
- 自动修复
	- 当容器失败时，会对容器进行重启
	- 当部署的node节点有问题时，会对容器进行重新部署和重新调度
	- 当容器未通过监控检查时，会关闭此容器直到容器可以正常运行，才会对外提供服务
- 水平扩展
	- 对容器进行规模扩大或裁剪
- 服务发现
	- 用户不需要额外的服务发现机制，基于`kubernertes`自身实现服务发现和负载均衡
- 滚动更新
	- 可以根据应用的变化对应用容器进行一次性或批量更新
- 版本回退
	- 可以对应用的容器进行历史版本即时回退
- 密钥和配置管理
	- 在不需要重新构建镜像的情况下，可以部署和更新密钥应用配置
- 存储编排
	- 自动实现存储系统挂载及应用
- 批处理
	- 提供一次性任务，定时任务

### 1.2架构组件

#### 1.2.1 kubernetes架构图

图一
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/k8s%E6%9E%B6%E6%9E%841.png)

图二
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/k8s%E6%9E%B6%E6%9E%842.png)


图三

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/k8s%E6%9E%B6%E6%9E%843.png)


#### 1.2.2kubernetes组件

-   **Master**：主控节点
    
    -   API Server：集群统一入口，以 restful 风格进行操作，同时交给 etcd 存储
        -   提供认证、授权、访问控制、API 注册和发现等机制
    -   scheduler：节点的调度，选择 node 节点应用部署
    -   controller-manager：处理集群中常规后台任务，一个资源对应一个控制器
    -   etcd：存储系统，用于保存集群中的相关数据
-   **Worker node**：工作节点
    
    -   Kubelet：master 派到 node 节点代表，管理本机容器
        -   一个集群中每个节点上运行的代理，它保证容器都运行在 Pod 中
        -   负责维护容器的生命周期，同时也负责 Volume(CSI) 和 网络 (CNI) 的管理
    -   kube-proxy：提供网络代理，负载均衡等操作
-   容器运行环境【**Container Runtime**】
    
    -   容器运行环境是负责运行容器的软件
    -   Kubernetes 支持多个容器运行环境：Docker、containerd、cri-o、rktlet 以及任何实现 Kubernetes CRI （容器运行环境接口） 的软件。
-   fluentd：是一个守护进程，它有助于提升集群层面日志

## 1.3 核心概念

1.  Pod
    
    -   Pod 是 K8s 中最小的单元
    -   一组容器的集合
    -   共享网络【一个 Pod 中的所有容器共享同一网络】
    -   生命周期是短暂的（服务器重启后，就找不到了）
2.  Volume
    
    -   声明在 Pod 容器中可访问的文件目录
    -   可以被挂载到 Pod 中一个或多个容器指定路径下
    -   支持多种后端存储抽象【本地存储、分布式存储、云存储】
3.  Controller
    
    -   确保预期的 pod 副本数量【ReplicaSet】
        
    -   无状态应用部署【Deployment】
        
        -   无状态就是指，不需要依赖于网络或者 ip
    -   有状态应用部署【StatefulSet】
        
        -   有状态需要特定的条件
    -   确保所有的 node 运行同一个 pod 【DaemonSet】
        
    -   一次性任务和定时任务【Job 和 CronJob】
        
4.  Deployment
    
    -   定义一组 Pod 副本数目，版本等
    -   通过控制器【Controller】维持 Pod 数目【自动回复失败的 Pod】
    -   通过控制器以指定的策略控制版本【滚动升级、回滚等】
5.  Service
    
    -   定义一组 pod 的访问规则
    -   Pod 的负载均衡，提供一个或多个 Pod 的稳定访问地址
    -   支持多种方式【ClusterIP、NodePort、LoadBalancer】
6.  Label
    
    -   label：标签，用于对象资源查询，筛选
7.  Namespace
    
    -   命名空间，逻辑隔离
    -   一个集群内部的逻辑隔离机制【鉴权、资源】
    -   每个资源都属于一个 namespace
    -   同一个 namespace 所有资源不能重复
    -   不同 namespace 可以资源名重复
8.  API
    
    -   我们通过 Kubernetes 的 API 来操作整个集群
    -   同时我们可以通过 kubectl 、ui、curl 最终发送 http + json/yaml 方式的请求给 API Server，然后控制整个 K8S 集群，K8S 中所有的资源对象都可以采用 yaml 或 json 格式的文件定义或描述


## k8s 网络检测工具

[尼古拉卡/网拍 - 码头工人图片 |码头工人中心 (docker.com)](https://hub.docker.com/r/nicolaka/netshoot)

```sh
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
```

## minikube

Minikube 是一个在本地运行 Kubernetes 集群的工具。由于 Minikube 运行在虚拟机或者本地的容器中，它的网络环境与宿主机的网络环境是隔离的。

为了将集群内部的服务暴露给宿主机或外部网络，需要进行端口转发。端口转发将宿主机的某个端口与集群内部的服务端口进行映射，这样宿主机就可以通过转发的端口访问集群内的服务。

在使用 `minikube service` 命令时，Minikube 会自动进行端口转发，将集群内的服务映射到宿主机上的一个端口。这样，您可以使用宿主机的 IP 地址和映射的端口来访问集群内的服务。

需要注意的是，如果您的网络环境发生变化或 Minikube 重新启动，端口转发可能会中断。因此，当需要长期使用端口转发时，建议将转发进程设置为后台运行，并且在需要时检查转发是否正常工作。


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230630111308.png)


在这里还需要将服务转发到绑定的端口30010 
```sh
kubectl port-forward svc/xc-simple-login --address 0.0.0.0 30010:8080
```

因为关闭终端或断开连接，转发仍然将会停止，这里将转发进程放在后台运行
```sh
nohup kubectl port-forward svc/xc-simple-login --address 0.0.0.0 30010:8080 > port-forward.log 2>&1 &
```