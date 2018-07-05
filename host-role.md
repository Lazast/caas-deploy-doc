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
>
> 注意这里唯一需要手工填入的地方

```bash
# master1 IP 具体IP地址以实际情况为准
CAAS_HOST_MASTER1=10.74.248.241
# master2 IP 具体IP地址以实际情况为准 
CAAS_HOST_MASTER2=10.74.248.242
# master3 IP 具体IP地址以实际情况为准 
CAAS_HOST_MASTER3=10.74.248.243

# storage1 IP 具体IP地址以实际情况为准 
CAAS_HOST_STORAGE1=10.74.248.249
# storage2 IP 具体IP地址以实际情况为准 
CAAS_HOST_STORAGE2=10.74.248.250

# loadbalance1 IP 具体IP地址以实际情况为准 
CAAS_HOST_LB1=10.74.248.247
# loadbalance1 IP 具体IP地址以实际情况为准 
CAAS_HOST_LB2=10.74.248.248

#node1 IP 具体IP地址以实际情况为准 
CAAS_HOST_NODE1=10.74.248.244
#node2 IP 具体IP地址以实际情况为准 
CAAS_HOST_NODE2=10.74.248.245
#node3 IP 具体IP地址以实际情况为准 
CAAS_HOST_NODE3=10.74.248.246

#harbor vip 具体IP地址以实际情况为准 
CAAS_VIP_HARBOR=10.74.248.251
#nfs vip 具体IP地址以实际情况为准 
CAAS_VIP_NFS=10.74.248.252
#loadbalance vip 具体IP地址以实际情况为准 
CAAS_VIP_LOADBALANCE=10.74.248.253
#mysql ldap vip 具体IP地址以实际情况为准 
CAAS_VIP_MYSQL_LDAP=10.74.248.254

#如果NODE节点数>3 
#NODE 的环境变量 KEY  为  CAAS_HOST_NODE4  CAAS_HOST_NODE5 CAAS_HOST_NODE6  以此类推
# 具体IP地址以实际情况为准 
CAAS_HOST_NODE4=
CAAS_HOST_NODE5=
CAAS_HOST_NODE6=
CAAS_HOST_NODE7=
CAAS_HOST_NODE8=
CAAS_HOST_NODE9=






# 生成环境变量配置文件
cat > ~/.bash_caas_env <<EOF
# master1 IP
export CAAS_HOST_MASTER1=${CAAS_HOST_MASTER1}
# master2 IP
export CAAS_HOST_MASTER2=${CAAS_HOST_MASTER2}
# master3 IP
export CAAS_HOST_MASTER3=${CAAS_HOST_MASTER3}

# storage1 IP
export CAAS_HOST_STORAGE1=${CAAS_HOST_STORAGE1}
# storage2 IP
export CAAS_HOST_STORAGE2=${CAAS_HOST_STORAGE2}

# loadbalance1 IP
export CAAS_HOST_LB1=${CAAS_HOST_LB1}
# loadbalance1 IP
export CAAS_HOST_LB2=${CAAS_HOST_LB2}

#node1 IP
export CAAS_HOST_NODE1=${CAAS_HOST_NODE1}
#node2 IP
export CAAS_HOST_NODE2=${CAAS_HOST_NODE2}
#node3 IP
export CAAS_HOST_NODE3=${CAAS_HOST_NODE3}

#harbor vip
export CAAS_VIP_HARBOR=${CAAS_VIP_HARBOR}
#nfs vip
export CAAS_VIP_NFS=${CAAS_VIP_NFS}
#loadbalance vip
export CAAS_VIP_LOADBALANCE=${CAAS_VIP_LOADBALANCE}
#mysql ldap vip
export CAAS_VIP_MYSQL_LDAP=${CAAS_VIP_MYSQL_LDAP}

#如果NODE节点数>3 
#NODE 的环境变量 KEY  为  CAAS_HOST_NODE4  CAAS_HOST_NODE5 CAAS_HOST_NODE6  以此类推
export CAAS_HOST_NODE4=${CAAS_HOST_NODE4}
export CAAS_HOST_NODE5=${CAAS_HOST_NODE5}
export CAAS_HOST_NODE6=${CAAS_HOST_NODE6}
export CAAS_HOST_NODE7=${CAAS_HOST_NODE7}
export CAAS_HOST_NODE8=${CAAS_HOST_NODE8}
export CAAS_HOST_NODE9=${CAAS_HOST_NODE9}


EOF


echo ". ~/.bash_caas_env" >> ~/.bashrc
source ~/.bashrc
```

## 

Next:  [Prerequistites](https://legacy.gitbook.com/book/jiulongzaitian/caas/edit#)

