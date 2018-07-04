# host roles

---

# 主机角色

## master 节点（**强制三台**）

> 例如：

```
10.74.248.241  （作为刚开始部署的数据存储包括 离线包）   
10.74.248.242     
10.74.248.243
```

## node节点\(至少两台\)

> 例如

```
10.74.248.244
10.74.248.245
10.74.248.246
```

## 负载均衡节点（强制两台）

> 例如

```
10.74.248.247
10.74.248.248
```

## 存储节点 （强制两台）

> 例如

```
10.74.248.249   10.74.248.250
```

## VIP （强制4个）

> 例如

```
CAAS_HARBOR_VIP=10.74.248.251
CAAS_NFS_VIP=10.74.248.252
CAAS_LOADBALANCE_VIP=10.74.248.253
CAAS_MYSQL_LDAP_VIP=10.74.248.254
```

## 生成环境变量

> 环境变量的值需要根据特定现场情况去设置
>
> 如果NODE节点数&gt;3 ,NODE 的环境变量 KEY  为  CAAS\_NODE4  CAAS\_NODE5 CAAS\_NODE6 ... 以此类推

```bash
{

cat > ~/bash_caas_env <<EOF
# master1 IP
CAAS_MASTER1=10.74.248.241
# master2 IP
CAAS_MASTER2=10.74.248.242
# master3 IP
CAAS_MASTER3=10.74.248.243

# storage1 IP
CAAS_STORAGE1=10.74.248.249
# storage2 IP
CAAS_STORAGE2=10.74.248.250

# loadbalance1 IP
CAAS_LB1=10.74.248.247
# loadbalance1 IP
CAAS_LB2=10.74.248.248

#node1 IP
CAAS_NODE1=10.74.248.244
#node2 IP
CAAS_NODE2=10.74.248.245
#node3 IP
CAAS_NODE3=10.74.248.246

#harbor vip
CAAS_HARBOR_VIP=10.74.248.251
#nfs vip
CAAS_NFS_VIP=10.74.248.252
#loadbalance vip
CAAS_LOADBALANCE_VIP=10.74.248.253
#mysql ldap vip
CAAS_MYSQL_LDAP_VIP=10.74.248.254

#如果NODE节点数>3 ,NODE 的环境变量 KEY  为  CAAS_NODE4  CAAS_NODE5 CAAS_NODE6 ... 以此类推
CAAS_NODE4=
CAAS_NODE5=
CAAS_NODE6=
CAAS_NODE7=
CAAS_NODE8=
CAAS_NODE9=
CAAS_NODE10=
CAAS_NODE11=
CAAS_NODE12=
CAAS_NODE13=
CAAS_NODE14=
CAAS_NODE15=
CAAS_NODE16=
CAAS_NODE17=
CAAS_NODE18=
CAAS_NODE17=
CAAS_NODE18=
CAAS_NODE19=
CAAS_NODE20=


EOF
}
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

Next:  [Prerequistites](https://legacy.gitbook.com/book/jiulongzaitian/caas/edit#)

