# host roles

---

划分不同主机角色

这里要充分考虑到 三种情况

1  管理，业务，存储 公用一个网卡

1 管理 一个网卡， 业务，存储公用一个网卡

1 管理一个网卡，业务一个网卡，存储一个网卡

## master

IP 信息 ，要设置为环境变量

```bash
#master1 管理网卡ip
MASTER1_MANAGER_IP=

# master1 业务网卡ip
MASTER1_BUSSINESS_IP=

#master1 存储网卡ip
MASTER1_STORAGE_IP=

#master1 管理网卡名称
MASTER1_MANAGER_INTERFACE=

#master1 业务网卡名称
MASTER1_BUSSINESS_INTERFACE=

#master1 存储网卡名称
MASTER1_STORAGE_INTERFACE=
```

## node

IP信息 要设置为变量

```bash
NODES
```

## 辅助组件node

IP 信息， 要设置为变量

## 统一配置文件

所有的信息全部保存在一个配置文件中，用环境变量形式，目的用来copy 到所有机器中



Next:  [docker](/docker.md)
