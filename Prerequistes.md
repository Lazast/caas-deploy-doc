# Prerequistes

---

安装caas 平台提前需要准备的工具

## 免密登陆

配置部署者本地机器，对 CAAS\_MASTER1  机器的ssh 免密登陆，要求本地机器是linux 环境

> 命令如下:

```
{
   #CAAS_MASTER1 的值 在上一篇文章中已经设置，查找相关的值，取代***
   CAAS_MASTER1=***

   ssh-keygen -t rsa -b 1024 -C "root"

   # 各种回车


   ssh-copy-id root@${CAAS_MASTER1}
}
```

## 

## 安装包

```
 yum install vim -y
```

## 离线安装包

## 

## 

## 

## 

## 

Next:  [docker](/docker.md)[ ](/host-role.md)

