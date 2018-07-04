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
CAAS_VIP_HARBOR=10.74.248.251
CAAS_VIP_NFS=10.74.248.252
CAAS_VIP_LOADBALANCE=10.74.248.253
CAAS_VIP_MYSQL_LDAP=10.74.248.254
```

## 生成环境变量

> 环境变量的值需要根据特定现场情况去设置
>
> 如果NODE节点数&gt;3 ,NODE 的环境变量 KEY  为  CAAS\_NODE4  CAAS\_NODE5 CAAS\_NODE6 ... 以此类推

```bash
{
#
cat > ~/.bash_caas_env <<EOF
# master1 IP
export CAAS_HOST_MASTER1=10.74.248.241
# master2 IP
export CAAS_HOST_MASTER2=10.74.248.242
# master3 IP
export CAAS_HOST_MASTER3=10.74.248.243

# storage1 IP
export CAAS_HOST_STORAGE1=10.74.248.249
# storage2 IP
export CAAS_HOST_STORAGE2=10.74.248.250

# loadbalance1 IP
export CAAS_HOST_LB1=10.74.248.247
# loadbalance1 IP
export CAAS_HOST_LB2=10.74.248.248

#node1 IP
export CAAS_HOST_NODE1=10.74.248.244
#node2 IP
export CAAS_HOST_NODE2=10.74.248.245
#node3 IP
export CAAS_HOST_NODE3=10.74.248.246

#harbor vip
export CAAS_VIP_HARBOR=10.74.248.251
#nfs vip
export CAAS_VIP_NFS=10.74.248.252
#loadbalance vip
export CAAS_VIP_LOADBALANCE=10.74.248.253
#mysql ldap vip
export CAAS_VIP_MYSQL_LDAP=10.74.248.254

#如果NODE节点数>3 
#NODE 的环境变量 KEY  为  CAAS_HOST_NODE4  CAAS_HOST_NODE5 CAAS_HOST_NODE6  以此类推
export CAAS_HOST_NODE4=
export CAAS_HOST_NODE5=
export CAAS_HOST_NODE6=
export CAAS_HOST_NODE7=
export CAAS_HOST_NODE8=
export CAAS_HOST_NODE9=


EOF


echo ". ~/.bash_caas_env" >> ~/.bashrc
source ~/.bashrc

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

