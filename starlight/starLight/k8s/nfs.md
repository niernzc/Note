## 服务端

安装`nfs-utils`

```shell
[root@localhost yum.repos.d]# yum -y install nfs-utils rpcbind
```

设置 NFS 服务开机启动

```shell
$ sudo systemctl enable rpcbind
$ sudo systemctl enable nfs
```

启动 NFS 服务

```shell
$ sudo systemctl start rpcbind
$ sudo systemctl start nfs
```

开启防火墙的 `rpc-bind`,`mountd`,`nfs`服务：

```shell
[root@localhost yum.repos.d]# sudo firewall-cmd --zone=public --permanent --add-service={rpc-bind,mountd,nfs}
success
[root@localhost yum.repos.d]# sudo firewall-cmd --reload
success
```

配置共享目录

```shell
[root@localhost /]# mkdir /nfs
[root@localhost /]# chmod 755 /nfs
[root@localhost /]# vi /etc/exports
[root@localhost /]# cat /etc/exports
/nfs 192.168.182.0/24(rw,sync,no_root_squash,no_all_squash)
[root@localhost /]# sudo systemctl restart nfs
```

## 客户端：

安装工具和配置服务(客户端无需开启防火墙相关端口，也无需开启NFS服务，因为不共享目录)

```shell
[root@localhost /]# yum -y install nfs-utils rpcbind
[root@localhost /]# sudo systemctl enable rpcbind
[root@localhost /]# sudo systemctl start rpcbind
```

客户端连接到NFS

```shell
[root@localhost /]# showmount -e 192.168.182.130
Export list for 192.168.182.130:
[root@localhost /]# mkdir /nfs
[root@localhost /]# mount -t nfs 192.168.182.130:/nfs /nfs
```

