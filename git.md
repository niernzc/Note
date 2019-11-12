### 工作流

本地仓库有git维护的三棵树组成

* 工作目录：持有实际文件
* 暂存区：缓存区域，临时保存改动
* HEAD：指向最后一次提交的结果

## 添加和提交

提出更改并添加到暂存区：

```shell
git add <filename>
git add *
```

使用如下命令实际提交改动：

```shell
git commit -m "提交信息"
```

此时改动已经提交到了HEAD，但没有到远端仓库

### 推送改动

改动已经在本地仓库的HEAD中，执行下面指令将这些改动提交到远端仓库

```shell
git push origin master
```

如果欸有克隆现有代码，并欲将仓库连接到某个远程服务器，可与使用如下命令：

```shell
git remote add origin <server>
```

