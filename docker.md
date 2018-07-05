# docker

---

## docker 配置

由于在上一篇文章中，我们已经通过ansible 自动全部安装完docker， 这里需要对docker 进行配置

> 在本地主机终端上，执行下面的命令， 获得所有主机列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_ |awk -F '=' '{print $2}'
```

> 注意，如果您有iterm 可以使用iterm 的终端广播功能
>
> 接下来 要顺序ssh登陆所有的打印出来的主机 ,都要执行后续的命令，

## 每台docker 配置

> ssh 登陆主机后， 执行下面的命令

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

> 我们需要







## 验证

Next:  [keepalived & haproxy](/keepalivedandhaproxy.md)

