# [Introducing Moby Project: a new open-source project to advance the software containerization movement](https://www.docker.com/blog/introducing-the-moby-project/)

docker推动软件容器化以来，整个生态都围绕者容器化发展，其构建容器的模型在不断演化

Moby项目是一个新的开源项目，旨在推动软件容器化运动。它提供了一个组件库，一个用于将他们集成到用户容器系统的框架和一个让开发者进行实验和交流意见的平台

### docker历史回顾

2013-2014:先驱者们开始使用容器并在一个整体的开源软件库（docker以及其他项目）中协同工作，来使工具成熟

![Docker开源](https://i1.wp.com/www.docker.com/blog/wp-content/uploads/2-2.png?resize=975%2C548&ssl=1)

2015-2016：容器开始大规模在云原生产品中使用，在这一阶段，用户社区迅速成长，docker将其生产模型演化成基于开放组件的方法。这样，我们可以同时增加创新和协作的表面积。docker从docker代码库中提取出了组件并进行快速创新，以便系统制造商在构建自己的容器系统时可以独立地重用它们：[runc](https://github.com/opencontainers/runc)，[HyperKit](https://github.com/docker/hyperkit)，[VPNKit](https://github.com/docker/vpnkit)，[SwarmKit](https://github.com/docker/swarmkit)，[InfraKit](https://github.com/docker/infrakit)，[containerd](https://github.com/containerd/containerd)等

![Docker开放式组件](https://i0.wp.com/www.docker.com/blog/wp-content/uploads/3-2.png?resize=975%2C548&ssl=1)

![Docker生产模型](https://i1.wp.com/www.docker.com/blog/wp-content/uploads/4-2.png?resize=975%2C548&ssl=1)

![白鲸计划](https://i0.wp.com/www.docker.com/blog/wp-content/uploads/5-2.png?resize=975%2C548&ssl=1)

moby包括：

1. 一个**库**集装箱后端组件（例如，一个低级别的建设者，测井设备，卷管理，网络，图像管理，containerd，SwarmKit，...）
2. 一个**框架，**用于将组件组装到独立的容器平台中，并提供用于为这些组件构建，测试和部署工件的工具。
3. 一个参考组件，称为**Moby Origin**，它是Docker容器平台的开放基础，以及使用Moby库或其他项目中各种组件的容器系统的示例。

Moby是为希望构建自己的基于容器的系统的系统构建者而设计的，而不是为可以使用Docker或其他容器平台的应用程序开发者而设计的。Moby项目的参与者可以从Docker派生的组件库中进行选择，也可以选择打包成容器的“自带组件”（BYOC），并可以在所有组件之间进行混合和匹配以创建自定义的容器系统。

 ***Docker is committed to using Moby as the upstream for the Docker Product.***