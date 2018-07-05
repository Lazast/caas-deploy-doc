# docker

---

## docker 配置

由于在上一篇文章中，我们已经通过ansible 自动全部安装完docker， 这里需要对docker 进行配置

> docker 的配置和分区设置 ，不同的角色主机，有不同的配置，需要单独配置

### master  配置

> 在本地主机终端上，执行下面的命令， 获得所有MASTER 节点列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_MASTER  |awk -F '=' '{print $2}'
```

1



1

1

1



### 

### 

### node  配置

在本地主机终端上，执行下面的命令， 获得所有Node 节点列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_NODE  |awk -F '=' '{print $2}'
```

1



1

1

1

### 存储  配置

在本地主机终端上，执行下面的命令， 获得所有存储 节点列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_STORAGE  |awk -F '=' '{print $2}'
```

1



1

1

1

### 负载均衡 配置

在本地主机终端上，执行下面的命令， 获得所有负载均衡 节点列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_LB  |awk -F '=' '{print $2}'
```



1



1

1

1





\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

ssh 登陆主机后， 执行下面的命令

```
lsblk -a
```

> 输出内容如下

```
# 下面的为输出内容
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   40G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 39.5G  0 part
  ├─centos-root 253:0    0 35.6G  0 lvm  /
  └─centos-swap 253:1    0  3.9G  0 lvm  [SWAP]
sdb               8:16   0  100G  0 disk
sdc               8:32   0  100G  0 disk
└─sdc1            8:33   0  100G  0 part /deploydata
sr0              11:0    1 1024M  0 rom
```

> 注意，需要分区的服务 有docker，openshift，（prometheus,  es 组件服务\)。
>
> 容量需求 从大大小排列： docker -&gt;
>
> 我们需要找到一个磁盘或者分区大于100G用于docker生产环境的direct-lvm存储配置，使用如下命令创建docker-vg

```
  pvcreate /dev/sdb
  vgcreate docker-vg /dev/sdb
```

## 验证

Next:  [keepalived & haproxy](/keepalivedandhaproxy.md)

