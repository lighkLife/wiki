# Kubernetes介绍

## Kubernetes 引入

### 作用
Kubernetes is an orchestrator of containerized cloud-native microservices apps.
Kubernetes deploys and manages applications that are packaged as containers and can easily scale, self-heal, and be updated.

Kubernetes 是`云原生` `微服务` 应用的 `编排器`。 通俗的说，Kubernetes 用来**部署**和**管理**那些打包成容器的应用，并且可以自动缩放、自动治理、自动更新应用。

```{note}
**云原生**

云原生应用具有云一样(cloud-like)的特性，比如自动缩放、自动治理、自动更新、回滚等等。<br>
只是简单的将一个应用部署在公共云上，并不能成为云原生。
```

Kubernetes 本质上就干两件事：

- 将云计算基础设施**抽象化**（比如AWS、阿里云）
- 将应用在云之间的**迁移简化**。


### 云 OS

- Linux 和 Windows 抽象化了单个服务器的资源，调度应用进程。
- K8S 抽象化了云服务器资源，调度微服务应用

因此，可以说 K8S 就是云服务的操作系统。

##
