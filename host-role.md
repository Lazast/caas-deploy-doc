# host roles

---

## 主机角色

### master 节点（**强制三台**）

> 请在本机linux 终端中输入，具体IP地址以当前实际情况为准

```
# master1 IP 具体IP地址以实际情况为准
CAAS_HOST_MASTER1=10.74.248.241
# master2 IP 具体IP地址以实际情况为准 
CAAS_HOST_MASTER2=10.74.248.242
# master3 IP 具体IP地址以实际情况为准 
CAAS_HOST_MASTER3=10.74.248.243
```

### node节点\(至少两台\)

> 请在本机linux 终端中输入，具体IP地址以当前实际情况为准

```
#node1 IP 具体IP地址以实际情况为准
CAAS_HOST_NODE1=10.74.248.244
#node2 IP 具体IP地址以实际情况为准
CAAS_HOST_NODE2=10.74.248.245
#node3 IP 具体IP地址以实际情况为准
CAAS_HOST_NODE3=10.74.248.246


#如果NODE节点数>3
#NODE 的环境变量 KEY 为 CAAS_HOST_NODE4 CAAS_HOST_NODE5 CAAS_HOST_NODE6 以此类推
# 具体IP地址以实际情况为准 若没有多余的node 请不输入
CAAS_HOST_NODE4=
CAAS_HOST_NODE5=
CAAS_HOST_NODE6=
CAAS_HOST_NODE7=
CAAS_HOST_NODE8=
CAAS_HOST_NODE9=
```

### 负载均衡节点（强制两台）

> 请在本机linux 终端中输入，具体IP地址以当前实际情况为准

```
# loadbalance1 IP 具体IP地址以实际情况为准 
CAAS_HOST_LB1=10.74.248.247
# loadbalance1 IP 具体IP地址以实际情况为准 
CAAS_HOST_LB2=10.74.248.248
```

### 存储节点 （强制两台）

> 请在本机linux 终端中输入，具体IP地址以当前实际情况为准

```bash
#storage1 IP 具体IP地址以实际情况为准
CAAS_HOST_STORAGE1=10.74.248.249
# storage2 IP 具体IP地址以实际情况为准 
CAAS_HOST_STORAGE2=10.74.248.250
```

### VIP （强制4个）

> 请在本机linux 终端中输入，具体IP地址以当前实际情况为准

```
#harbor vip 具体IP地址以实际情况为准 
CAAS_VIP_HARBOR=10.74.248.251

#nfs vip 具体IP地址以实际情况为准 
CAAS_VIP_NFS=10.74.248.252

#loadbalance vip 具体IP地址以实际情况为准 
CAAS_VIP_LOADBALANCE=10.74.248.253

#mysql ldap vip 具体IP地址以实际情况为准 
CAAS_VIP_MYSQL_LDAP=10.74.248.254
```

### 容器网络规划

> 请根据部署环境的实际情况，对容器的网络进行规划，可参考下面的例子。

```
# 系统中主机ip的掩码为9位，系统支持的最大主机数为2^9=512
CAAS_OSM_HOST_SUBNET_LENGTH=9
# 每个主机上的pod ip掩码位数为18-9=9，即每台主机上最多有2^9=512个pod的ip
CAAS_OSM_CLUSTER_NETWORK_CIDR=10.128.0.0/18
```

## 域名

> 请在本机linux 终端中输入，具体地址以当前实际情况为准，具体泛域名后缀\(example.com\)以实际情况为准

```
CAAS_DOMAIN_LDAP=ldap.caas.example.com
CAAS_DOMAIN_HARBOR=harbor.caas.example.com
CAAS_DOMAIN_OS_CONSOLE=os-console.caas.example.com
CAAS_DOMAIN_ALERT=alertmanager.caas.example.com
CAAS_DOMAIN_PROM=prometheus.caas.example.com
CAAS_DOMAIN_GRAFANA=grafana.caas.example.com
CAAS_DOMAIN_ES=es.caas.example.com
CAAS_DOMAIN_PORTAL=portal.caas.example.com
CAAS_DOMAIN_PORTAL_API=portalapi.caas.example.com
CAAS_DOMAIN_PAN=caas.example.com
```

## 生成环境变量

> 环境变量的值需要根据特定现场情况去设置
>
> 如果NODE节点数&gt;3 ,NODE 的环境变量 KEY  为  CAAS\_NODE4  CAAS\_NODE5 CAAS\_NODE6 ... 以此类推
>
> 注意这里唯一需要手工填入的地方

```bash
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

#harbor vip
export CAAS_VIP_HARBOR=${CAAS_VIP_HARBOR}
#nfs vip
export CAAS_VIP_NFS=${CAAS_VIP_NFS}
#loadbalance vip
export CAAS_VIP_LOADBALANCE=${CAAS_VIP_LOADBALANCE}
#mysql ldap vip
export CAAS_VIP_MYSQL_LDAP=${CAAS_VIP_MYSQL_LDAP}

#node1 IP
export CAAS_HOST_NODE1=${CAAS_HOST_NODE1}
#node2 IP
export CAAS_HOST_NODE2=${CAAS_HOST_NODE2}
#node3 IP
export CAAS_HOST_NODE3=${CAAS_HOST_NODE3}


#如果NODE节点数>3 
#NODE 的环境变量 KEY  为  CAAS_HOST_NODE4  CAAS_HOST_NODE5 CAAS_HOST_NODE6  以此类推
# 若没有多余的node 请注释掉
export CAAS_HOST_NODE4=${CAAS_HOST_NODE4}
export CAAS_HOST_NODE5=${CAAS_HOST_NODE5}
export CAAS_HOST_NODE6=${CAAS_HOST_NODE6}
export CAAS_HOST_NODE7=${CAAS_HOST_NODE7}
export CAAS_HOST_NODE8=${CAAS_HOST_NODE8}
export CAAS_HOST_NODE9=${CAAS_HOST_NODE9}

# 域名

export CAAS_DOMAIN_LDAP=${CAAS_DOMAIN_LDAP}
export CAAS_DOMAIN_HARBOR=${CAAS_DOMAIN_HARBOR}
export CAAS_DOMAIN_OS_CONSOLE=${CAAS_DOMAIN_OS_CONSOLE}
export CAAS_DOMAIN_ALERT=${CAAS_DOMAIN_ALERT}
export CAAS_DOMAIN_PROM=${CAAS_DOMAIN_PROM}
export CAAS_DOMAIN_GRAFANA=${CAAS_DOMAIN_GRAFANA}
export CAAS_DOMAIN_ES=${CAAS_DOMAIN_ES}
export CAAS_DOMAIN_PORTAL=${CAAS_DOMAIN_PORTAL}
export CAAS_DOMAIN_PORTAL_API=${CAAS_DOMAIN_PORTAL_API}
export CAAS_DOMAIN_PAN=${CAAS_DOMAIN_PAN}
export CAAS_OSM_HOST_SUBNET_LENGTH=${CAAS_OSM_HOST_SUBNET_LENGTH}
export CAAS_OSM_CLUSTER_NETWORK_CIDR=${CAAS_OSM_CLUSTER_NETWORK_CIDR}


EOF
```

> 环境变量配置文件 生成完后，请用vim 打开，再次确认一下 配置是否正确
>
> 并注释掉没有用的 CAAS\_HOST\_NODE\*\*\* 相关的环境变量，请确保NODE 相关环境变量是正确的

```
vim ~/.bash_caas_env


######  配置文件内容  ######
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

# 域名
export CAAS_DOMAIN_LDAP=ldap.caas.example.com
export CAAS_DOMAIN_HARBOR=harbor.caas.example.com
export CAAS_DOMAIN_OS_CONSOLE=os-console.caas.example.com
export CAAS_DOMAIN_ALERT=alertmanager.caas.example.com
export CAAS_DOMAIN_PROM=prometheus.caas.example.com
export CAAS_DOMAIN_GRAFANA=grafana.caas.example.com
export CAAS_DOMAIN_ES=es.caas.example.com
export CAAS_DOMAIN_PORTAL=portal.caas.example.com
export CAAS_DOMAIN_PORTAL_API=portalapi.caas.example.com
export CAAS_DOMAIN_PAN=${CAAS_DOMAIN_PAN}
export CAAS_OSM_HOST_SUBNET_LENGTH=${CAAS_OSM_HOST_SUBNET_LENGTH}
export CAAS_OSM_CLUSTER_NETWORK_CIDR=${CAAS_OSM_CLUSTER_NETWORK_CIDR}



#如果NODE节点数>3
#NODE 的环境变量 KEY  为  CAAS_HOST_NODE4  CAAS_HOST_NODE5 CAAS_HOST_NODE6  以此类推

#这些环境变量没用到，注释掉
#export CAAS_HOST_NODE4=
#export CAAS_HOST_NODE5=
#export CAAS_HOST_NODE6=
#export CAAS_HOST_NODE7=
```

> export 环境变量

```bash
echo ". ~/.bash_caas_env" >> ~/.bashrc
source ~/.bashrc
```

> copy 环境变量 文件 到  master1

```
scp ~/.bash_caas_env root@$CAAS_HOST_MASTER1:~
ssh root@$CAAS_HOST_MASTER1 "sed -i '/^\.\ ~\/\.bash_caas_env/d' ~/.bashrc"
ssh root@$CAAS_HOST_MASTER1 "echo '. ~/.bash_caas_env' >> ~/.bashrc"
```

Next:  [Prerequistites](https://legacy.gitbook.com/book/jiulongzaitian/caas/edit#)

