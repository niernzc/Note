总架构图：

![docker-1-2](http://blog.daocloud.io/wp-content/uploads/2014/12/001_docker_architecture.jpg)

## client:

docker架构中用户与docker daemon建立通信的客户端

## docker daemon：

* 接受并处理client发送来的请求
* 管理所有容器

### server：

docker server在架构服务中专门服务于docker client，其功能使接受并调度docker client的请求

### Engine：

 运行引擎，同时也是docker运行的核心模块，存储着大量容器核心并管理者docker大部分job的执行。docker的大部分任务都需要Engine的协助，并通过Engine匹配相应的job完成job的执行

### Job：

Engine内部最基本的工作执行单元，其每一项工作都会呈现为一个Job。例如，在docker容器内部运行一个进程、创建一个新的容器。

### Docker Registry

存储容器镜像的仓库。docker在运行过程中，有三种情况可能与Docker registry通信，分别是搜索镜像、下载镜像和上传镜像，对应的job分别为search、pull、push。

### Graph

容器镜像的保管者，不论是docker下载的镜像，还是docker构建的镜像，均由Graph统一管理。对docker而言，同一类型的镜像属于同一个repository；而统一各repository下的镜像则会因tag存在差异而不同，如ubuntu这个repository下有tag为12.04的镜像，也有tag为14.04的镜像

### Driver

Docker架构中的驱动模块，通过Driver驱动，Docker可以实现对Docker容器运行环境的定制，定制的维度主要有网络环境、存储方式以及容器执行方式，主要包含三类驱动：

* graphdriver：主要用于完成容器镜像的管理
* networkdriver：完成对容器网络环境的配置，其中包括Docker Daemon启动时为Docker环境创建的网桥；Docker容器创建前为其分配相应的网络接口资源；以及为Docker容器分配 IP，端口并与宿主机做NAT端口映射，防火墙策略等

