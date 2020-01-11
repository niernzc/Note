#### ubuntu虚拟机中使用NAT，ens33没有ip

```shell
sudo dhclient ens33
```

### ubuntu 开机运行脚本

* Ubuntu开机执行/etc/rc.local文件中的脚本
* 先将脚本复制到/etc/init.d/目录下

#### vmware 共享文件夹

```shell
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o subtype=vmhgfs-fuse,allow_other #将所有共享文件夹挂载到/mnt/hgfs
```

```shell
nzc@ubuntu:/$ sudo vmhgfs-fuse .host:/ /mnt/hgfs -o subtype=vmhgfs-fuse,allow_other,nonempty
nzc@ubuntu:/mnt/hgfs$ sudo ln -sf /mnt/hgfs/starLightSharefolder/ /
nzc@ubuntu:/mnt/hgfs$ ls /
bin    etc             lib64       proc  srv                   usr
boot   home            lost+found  root  starLightSharefolder  var
cdrom  initrd.img      media       run   swapfile              vmlinuz
data   initrd.img.old  mnt         sbin  sys                   vmlinuz.old
dev    lib             opt         snap  tmp
```

