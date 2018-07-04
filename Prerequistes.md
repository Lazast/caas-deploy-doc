# Prerequistes

---

安装caas 平台提前需要准备的工具

# 免密登录

> 本地主机行执行如下命令,对所有主机免密登录

```
hosts=$(env |grep CAAS_HOST_ |awk -F '=' '{print $2}')

if [ ! -f ~/.ssh/id_rsa.pub ]; then

    ssh-keygen -t rsa -b 1024 -C "root"

fi

for h in $hosts; do

    ssh-copy-id root@$h

done

for h in $hosts; do

    scp ~/.bash_caas_env root@$h:~

done

for h in $hosts; do

    scp ~/.bash_caas_env root@$h:~

done

for h in $hosts; do

    ssh root@$h "  sed -i '/^\.\ ~\/\.bash_caas_env/d' ~/.bashrc  ; echo \". ~/.bash_caas_env\" >> ~/.bashrc"

done
```

# 安装离线包

## 登陆CAAS\_MASTER1

```
{
ssh root@${CAAS_MASTER1}
}
```

## 配置离线安装包

> 在CAAS\_MASTER1 上 ，找一块分区，分区大小必须 &gt; 50G， 若没有大于50G的分区，请联系客户或相关人员，增加盘或者划分去
>
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

> 请执行以下命令 获得&gt;50G 的分区目录

```
{
validdata=$(df -m |sed 1d |sort -rn -k2 |awk '{if($2>50000) print $6}'  | head -1)
echo $validdata
if [ "$validdata" == "" ]; then
   echo "没有找到>50G的分区，无法执行后续的操作，请联系相关人员 ..."
   exit 1
else
   echo "找到>50G的分区目录: $validdata"
   offlinedata=$validdata
   echo "offlinedata=$validdata" >> ~/.bashrc
   mkdir -p $offlinedata
   exit 1
fi


}
```

> 将本地caas-offline.tar 文件scp 到 CAAS\_MASTER1 机器

```
{
scp ./caas-offline.tar root@${CAAS_MASTER1}:~

}
```

> 重新登录CAAS\_MASTER1机器

```
{
# 重新登录CAAS_MASTER1
ssh root@${CAAS_MASTER1}
}
```

> 解压离线文件

```
tar xvf ~/caas-offline.tar -C $offlinedata

cd $offlinedata/caas-offline/cent7.2

# 启动 http 服务


nohup python -m SimpleHTTPServer 38888 &

# 配置iptables 规则，其实能访问
iptables -I INPUT -p tcp  --dport 38888 -j ACCEPT
```



# 配置yum 源



## 

## 

## 

## 

## 

Next:  [docker](/docker.md)[ ](/host-role.md)

