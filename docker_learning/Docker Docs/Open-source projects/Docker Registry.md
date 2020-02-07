#### what it is

Registry是一个无状态的、高可扩展的服务端应用，它可以存储并允许你发布自己的Docker镜像

#### Why use it

* 控制镜像的存储位置
* 拥有自己的镜像分发管道
* 将镜像存储和分发整合到本地的开发工作流中



## About Registry

一个Registry是一个存储和内容传输系统，保持命名的Docker镜像，并允许访问其带有不同tag的版本。用户可以通过`push`和`pull`与之交互。

镜像的存储自身被委托给驱动程序，默认的存储驱动是本地的posix文件系统，其适合于开发和小型部署。额外的基于云的存储驱动，比如`S3`、`Microsoft Azure`，`OpenStack Swift`以及`Aliyun OSS`同样支持，想要使用其他存储后端也可以通过实现[Storage API](https://docs.docker.com/registry/storage-drivers/)，来写自己的驱动

因为对主机上的镜像的安全访问是最重要的（paramount），Registry原生支持TLS和基本认证，Github仓库中有关于高级认证和授权方法的其他信息。只有非常大型或者公共部署才需要在这些方面对Registry进行扩展

最后，Registry附带了一个强大的[通知系统](https://docs.docker.com/registry/notifications/)，可以响应活动而调用webhooks，并且具有广泛的日志记录和报告功能，这对于想要收集指标的大型安装非常有用。

#### 理解镜像命名

镜像命名用于在docker命令中反映其来源

* `docker pull ubuntu`指示Docker从官方Docker Hub 提取命名的镜像。这只是较长`docker pull docker.io/library/ubuntu`命令的快捷方式
* `docker pull myregistrydomain:port/foo/bar`指示Docker寻找位于的registry`myregistrydomain:port`以查找镜像`foo/bar`

#### 用例 

运行自己的Registry是与CI/CD系统集成并对其进行补充的绝佳方案。在典型的工作流程中，对源版本控制系统的提交将触发在CI系统上的构建，若构建成功，则将新镜像推送到Registry。然后，来自Registry的通知将触发在staging environment上的部署，或者提醒其他系统当前有一个新的镜像可用。当你想要在一个大型集群或机器上部署一个新的镜像时，这同样是一个非常重要的组件。最后这是在一个与外界隔离的网络中分发镜像的最好方式。

