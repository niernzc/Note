### Docker数据管理

通常情况下，容器中创建出来的文件都被保存在一个可写容器层中，意味着：

* 容器退出时数据不会保存，且难以将数据迁移出容器
* 将数据写入一个容器可写层意味着需要一个存储驱动来管理文件系统

docker提供了两种方式来将文件存储到宿主机，以使这些文件在容器退出后还能保存：`volume`和`bind mount`，以及linux上的`tmpfs mount`和windows上的`nameed pipe`。

无论选取哪种挂载方式，数据在容器中看起来都是一样的

