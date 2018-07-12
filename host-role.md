# resource planning

---

# 资源规划

本节对caas部署时的各种资源进行规划

**下面的操作全部在部署机（本文档中部署机为CAAS\_HOST\_MASTER1，即master节点的第一台机器）上执行**

> 生成规划文件

```
touch ~/.bash_caas_env
```

根据平台的实际规划，将下面的值进行替换并写入文件~/.bash\_caas\_env

## 主机角色

### master 节点（**强制三台**）

> 具体IP地址以当前实际情况为准

```
# master1 IP 具体IP地址以实际情况为准
export CAAS_HOST_MASTER1=10.74.248.241
# master2 IP 具体IP地址以实际情况为准 
export CAAS_HOST_MASTER2=10.74.248.242
# master3 IP 具体IP地址以实际情况为准 
export CAAS_HOST_MASTER3=10.74.248.243
```

### node节点\(至少两台\)

> 具体IP地址以当前实际情况为准

```
#node1 IP 具体IP地址以实际情况为准
export CAAS_HOST_NODE1=10.74.248.244
#node2 IP 具体IP地址以实际情况为准
export CAAS_HOST_NODE2=10.74.248.245
#node3 IP 具体IP地址以实际情况为准
export CAAS_HOST_NODE3=10.74.248.246


#如果NODE节点数>3
#NODE 的环境变量 KEY 为 CAAS_HOST_NODE4 CAAS_HOST_NODE5 CAAS_HOST_NODE6 以此类推
# 具体IP地址以实际情况为准 若没有多余的node 请不输入
export CAAS_HOST_NODE4=
export CAAS_HOST_NODE5=
export CAAS_HOST_NODE6=
export CAAS_HOST_NODE7=
export CAAS_HOST_NODE8=
export CAAS_HOST_NODE9=
```

### 负载均衡节点（强制两台）

> 具体IP地址以当前实际情况为准

```
# loadbalance1 IP 具体IP地址以实际情况为准 
export CAAS_HOST_LB1=10.74.248.247
# loadbalance1 IP 具体IP地址以实际情况为准 
export CAAS_HOST_LB2=10.74.248.248
```

### 存储节点 （强制两台）

> 具体IP地址以当前实际情况为准

```bash
#storage1 IP 具体IP地址以实际情况为准
export CAAS_HOST_STORAGE1=10.74.248.249
# storage2 IP 具体IP地址以实际情况为准 
export CAAS_HOST_STORAGE2=10.74.248.250
```

### VIP （强制4个）

> 具体IP地址以当前实际情况为准

```
#harbor vip 具体IP地址以实际情况为准 
export CAAS_VIP_HARBOR=10.74.248.251

#nfs vip 具体IP地址以实际情况为准 
export CAAS_VIP_NFS=10.74.248.252

#loadbalance vip 具体IP地址以实际情况为准 
export CAAS_VIP_LOADBALANCE=10.74.248.253

#mysql ldap vip 具体IP地址以实际情况为准 
export CAAS_VIP_MYSQL_LDAP=10.74.248.254
```

### 容器网络规划

> 请根据部署环境的实际情况，对容器的网络进行规划，可参考下面的例子。

```
# 系统中主机ip的掩码为9位，系统支持的最大主机数为2^9=512
export CAAS_OSM_HOST_SUBNET_LENGTH=9
# 每个主机上的pod ip掩码位数为18-9=9，即每台主机上最多有2^9=512个pod的ip
export CAAS_OSM_CLUSTER_NETWORK_CIDR=10.128.0.0/18
```

## 域名

> 具体地址以当前实际情况为准，具体泛域名后缀\(example.com\)以实际情况为准

```
export CAAS_DOMAIN_LDAP=ldap.caas.example.com
export CAAS_DOMAIN_HARBOR=harbor.caas.example.com
export CAAS_DOMAIN_OS_CONSOLE=os-console.caas.example.com
export CAAS_DOMAIN_ALERT=alertmanager.caas.example.com
export CAAS_DOMAIN_PROM=prometheus.caas.example.com
export CAAS_DOMAIN_GRAFANA=grafana.caas.example.com
export CAAS_DOMAIN_ES=es.caas.example.com
export CAAS_DOMAIN_PORTAL=portal.caas.example.com
export CAAS_DOMAIN_PORTAL_API=portalapi.caas.example.com
export CAAS_DOMAIN_PAN=caas.example.com
```

### 容器镜像TAG

> 下面 的参数是caas portal部署时使用的镜像tag，默认值如下。当caas portal有版本更新时，请根据实时的情况进行更新

```
# omp镜像的tag
export CAAS_VAR_OMP_IMAGE_TAG=v1
# agamaha镜像的tag
export CAAS_VAR_AGAMAHA_IMAGE_TAG=v1
# harpoxy镜像的tag
export CAAS_VAR_HAPROXY_IMAGE_TAG=v1
# hcenter镜像的tag
export CAAS_VAR_HCENTER_IMAGE_TAG=v1
# muddle镜像的tag
export CAAS_VAR_MUDDLE_IMAGE_TAG=v1
# redis镜像的tag
export CAAS_VAR_REDIS_IMAGE_TAG=v1
```



\#\#\#\#\#\#\#\#\#\#\#\#\#caas 资源规划至此结束\#\#\#\#\#\#\#\#\#

## 使配置生效

> 完成caas资源的规划，保存并退出文件~/.bash\_caas\_env， 在linux终端中执行以下命令

```bash
echo ". ~/.bash_caas_env" >> ~/.bashrc
source ~/.bashrc
```



Next:  [Prerequistites](https://legacy.gitbook.com/book/jiulongzaitian/caas/edit#)

