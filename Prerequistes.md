# Prerequistes

---

安装caas 平台提前需要准备的工具

## 对MASTER1免密登陆

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

## 登陆CAAS\_MASTER1

```
{
ssh root@${CAAS_MASTER1}
}
```

## 配置离线安装包

> 在CAAS\_MASTER1 上 ，找一块分区，分区大小必须 &gt; 50G， 若没有大于50G的分区，请联系客户或相关人员，增加盘或者划分去



> 分区查看命令  （Avail）

```
df -h


Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   36G  2.3G   34G   7% /
devtmpfs                 7.8G     0  7.8G   0% /dev
tmpfs                    7.8G     0  7.8G   0% /dev/shm
tmpfs                    7.8G  8.6M  7.8G   1% /run
tmpfs                    7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/sda1                497M  124M  373M  25% /boot
tmpfs                    1.6G     0  1.6G   0% /run/user/0
/dev/sdc1                100G   33M  100G   1% /deploydata
```

> 请执行以下命令

```
{
validdata=$(df -m |sed 1d |sort -rn -k2 |awk '{if($2>50000) print $6}'  | head -1)
echo $validdata
if [ "$validdata" == "" ]; then
   echo "没有找到>50G的分区，无法执行后续的操作，请联系相关人员 ..."
else
   echo "找到>50G的分区目录: $validdata"
   offlinedata="$validdata/caas"
   mkdir -p $offlinedata
fi



}
```

> 我们选择/deploydata 目录作为 离线安装包的存储目录

## 离线安装包

## 

## 

## 

## 

## 

Next:  [docker](/docker.md)[ ](/host-role.md)

