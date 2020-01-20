cAdviser可以搜集一台机器上所有运行的容器信息，还提供基础查询界面和http接口，方便其他组件如Prometheus进行抓取，或cAdviser + influxdb + grafna搭配使用

## [readme](https://github.com/google/cadvisor)：

cAdviser使用户能够了解其运行中的容器的资源使用情况和性能特征。它会运行一个daemon来收集、聚合、处理并输出运行中容器的信息。特别的，对于每一个容器，它会保留其资源隔离参数、历史资源使用量和网络统计信息。此数据按容器和机器范围导出

