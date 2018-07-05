# docker

---

## docker 配置

由于在上一篇文章中，我们已经通过ansible 自动全部安装完docker， 这里需要对docker 进行配置

> docker 的配置和分区设置 ，不同的角色主机，有不同的配置，需要单独配置

### master  和 Node 配置

> 在本地主机终端上，执行下面的命令， 获得所有MASTER 节点列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_MASTER  |awk -F '=' '{print $2}'
env |grep CAAS_HOST_NODE  |awk -F '=' '{print $2}'
```

> ssh 登陆每台master 和 node 主机， 输入下面命令 查看磁盘 和块设备

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
> 假设只有一个额外的数据盘 sdb，我们需要将此数据盘分为三个分区，分别给docker，openshift，和（数据盘），分区大小为3：3：4。 2T以下用fdisk 命令做分区， 2T以上用parted 命令做分区。
>
> docker 分区不需要挂载目录
>
> openshift 分区（需要格式化 使用  mkfs.xfs 命令格式化）需要挂载到  /var/lib/origin/openshift.local.volumes 目录,  使用mount -t xfs   命令
>
> 数据分区 （需要格式化 使用  mkfs.xfs 格式化 ）需要挂载到 /caas\_data/， 使用mount  -t xfs  命令
>
> 最后修改 /etc/fstab 文件，添加开机挂载目录，将openshift 和 数据分区设置为开机自动挂载
>
> 假设给docker 使用的分区为 /dev/sdb1, 请执行下面命令 ，创建vg

```
  pvcreate /dev/sdb1
  vgcreate docker-vg /dev/sdb1
```

### 存储  配置

在本地主机终端上，执行下面的命令， 获得所有存储 节点列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_STORAGE  |awk -F '=' '{print $2}'
```



ssh 登陆每台存储 主机， 输入下面命令 查看磁盘 和块设备

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

> 注意，需要分区的服务 有NFS，数据盘。
>
> 假设只有一个额外的数据盘 sdb，我们需要将此数据盘分为两个分区，分别给NFS，数据盘，分区大小为6：4。 2T以下用fdisk 命令做分区， 2T以上用parted 命令做分区。
>
> 如果有两个盘，一个盘给NFS 另一个给数据盘
>
> NFS分区不需要挂载目录

> 数据分区 （需要格式化 使用  mkfs.xfs 格式化 ）需要挂载到 /caas\_data/， 使用mount  -t xfs  命令
>
> 最后修改 /etc/fstab 文件，添加开机挂载目录，将 数据分区设置为开机自动挂载
>
> 假设给NFS 使用的分区为 /dev/sdb1, 请执行下面命令 ，创建vg



```
vgcreate  vg-paas /dev/sdb1 /dev/xxx（按照实际磁盘数量和策略决定）
mkdir /nfs
```

### 

Next:  [keepalived & haproxy](/keepalivedandhaproxy.md)

