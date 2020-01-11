通过starlight提交任务之后容器没有启动，查看pod状态：

```shell
[root@master ~]# kubectl get pod -n 1
NAME                                          READY   STATUS    RESTARTS   AGE
wa02cc3ff-45a4-4a54-b668-175aa6157f21-zjk77   0/1     Pending   0          3h6m
wfc2d8836-39e6-46a5-921e-58c61756e95c-wspt5   0/1     Pending   0          16s
```

```shell
[root@master ~]# kubectl describe pod wfc2d8836-39e6-46a5-921e-58c61756e95c-wspt5 -n 1
Name:           wfc2d8836-39e6-46a5-921e-58c61756e95c-wspt5
Namespace:      1
'''
Node-Selectors:  partition=
'''
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/2 nodes are available: 2 node(s) didn't match node selector.
```

找到Node-Selector这一项，非空，可能是这个原因，因为我貌似没有给node类似的‘label’

### 给node打上label

```shell
kubectl label nodes <node-name> <label-key>=<label-value>
```



