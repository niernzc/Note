使用共享存储，而非直接在本地存储镜像



5.2 VMstore 整合

AUFS的COW没用起来，Slacker采用共享存储，理论上可在daemon和registries之间采用COW,VMstore提供了相关的COW功能，snapshot、clone，前者为一个NFS文件创建一个只读的快照。（文件级别的快照和克隆可以用于高效的日志、去重等操作？？？）

在Slacker中，层次被扁平化了，因此通过Diff（A，B）可以直接得到B，而无需再在A的基础上通过其返回构造出B



5.3 优化快照和克隆

镜像包含许多层。块级别的COW相对于文件级别的COW又性能优势，因为遍历映射指数比迭代路径要更简单

然而，多层的镜像对于Slacker带来一个挑战，Slacker的镜像是扁平化的，所以仅需挂载一层便可提供容器使用的整个文件系统，但是docker不知道，会对所有层拉一遍
解决办法：没有修改Docker框架来限制其调用存储驱动。在Slacker中，pull的主要花销并非在网络传输，而在clone（用于实现ApplyDiff->pull），因此，Slacker没有将每一个层表现为一个NFS文件，而是将它们表现为snapshotID的一段元数据，ApplyDiff将仅仅设置其元数据而非直接进行cloning操作，而当Docker在哪一层调用Get时，再在挂载那个点之前进行真实的clone

同时使用snapshot ID进行快照缓存，Slacker在每一次的clone之后创建一个快照以实现Create，其功能是在创建一个层的逻辑拷贝，create：对clone后快照，applydiff：修改新快照元数据。