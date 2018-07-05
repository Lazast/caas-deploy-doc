# Prerequistes

---

安装caas 平台提前需要准备的工具

## 免密登录

> 本地主机行执行如下命令,对所有主机免密登录

```bash
hosts=$(env |grep CAAS_HOST_ |awk -F '=' '{print $2}')

if [ ! -f ~/.ssh/id_rsa.pub ]; then
    ssh-keygen -t rsa -b 1024 -C "root"
fi

for h in $hosts; do
    ssh-copy-id root@$h
done
```

## 安装离线包

### 登陆CAAS\_MASTER1

```bash
ssh root@${CAAS_HOST_MASTER1}
```

### 配置离线安装包

> 在CAAS\_HOST\_MASTER1 上 ，找一块分区，分区大小必须 &gt; 50G， 若没有大于50G的分区，请联系客户或相关人员，增加盘或者划分去
>
> 分区查看命令  （Avail）

```bash
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

```bash
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
```

> 将本地caas-offline.tar 文件scp 到 CAAS\_MASTER1 机器

```bash
scp ./caas-offline.tar root@${CAAS_HOST_MASTER1}:~
```

> 重新登录CAAS\_MASTER1机器

```bash
# 重新登录CAAS_MASTER1
ssh root@${CAAS_HOST_MASTER1}
```

> 解压离线文件

```bash
tar xvf ~/caas-offline.tar -C $offlinedata

cd $offlinedata/caas-offline/cent7.2

# 启动 http 服务


nohup python -m SimpleHTTPServer 38888 &

# 配置iptables 规则，其实能访问
iptables -I INPUT -p tcp  --dport 38888 -j ACCEPT
```

# 

## master1配置ansible

> 配置master1 到其他主机免密登陆

```bash
hosts=$(env |grep CAAS_HOST_ |awk -F '=' '{print $2}')

if [ ! -f ~/.ssh/id_rsa.pub ]; then
    ssh-keygen -t rsa -b 1024 -C "root"
fi

for h in $hosts; do
    ssh-copy-id root@$h
    # 注意 这里会让用户输入 yes 和用户名 密码
done
```

> 配置本地yum 源

```bash
cd /etc/yum.repos.d
mkdir back
mv *.repo back/

cat > caas.repo << EOF

[caas]
name=caas-offline-local
failovermethod=priority
baseurl=http://$CAAS_HOST_MASTER1:38888/
#mirrorlist=http://mirrorlist.centos.org/?release=&arch=&repo=os
gpgcheck=0
#gpgkey=http://mirrors.haihangyun.com/centos/RPM-GPG-KEY-CentOS-7
EOF

cp caas.repo /tmp/caas.repo
```

> 配置ansible 配置文件

```bash
cd $offlinedata/caas-offline
mkdir install
cd  install

env |grep CAAS_HOST_MASTER |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/masters
env |grep CAAS_HOST_NODE |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/nodes
env |grep CAAS_HOST_LB |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/lbs
env |grep CAAS_HOST_STORAGE |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/storage

cat > ansible_hosts <<EOF
[dockers:children]
masters
nodes
storages
lbs
[masters]
$(cat /tmp/masters)
[nodes]
$(cat /tmp/nodes)
[lbs]
$(cat /tmp/lbs)
[storages]
$(cat /tmp/storage)
EOF
```

> 创建hosts 附加文件

```bash
env |grep CAAS_HOST_MASTER |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" "arrays[3]}}' > extra_hosts
env |grep CAAS_HOST_NODE |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" "arrays[3]}}' >> extra_hosts
env |grep CAAS_HOST_LB |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" "arrays[3]}}' >> extra_hosts
env |grep CAAS_HOST_STORAGE |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" "arrays[3]}}' >> extra_hosts
```

> 查看文件

```
ls

# ansible_hosts  extra_hosts
```

> 安装ansible

```bash
yum install ansible -y
```

> 生成ansible 配置文件， 执行ansible 命令

```
cat > prepare.yaml << EOF

---
- hosts: all
  tasks:
    - name: bak prepare yum repo - dir create
      file: path=/etc/yum.repos.d/caas_bak state=directory
    - name: bak prepare yum repo - bak repo
      shell: mv -f /etc/yum.repos.d/*.repo /etc/yum.repos.d/caas_bak/
    - name: copy caas repo to all host
      copy: src=/tmp/caas.repo dest=/etc/yum.repos.d/ force=true
    - name: yum update
      shell: yum clean all && yum makecache

    - name: copy caas host resolve to all hosts
      copy: src=./extra_hosts dest=/tmp/extra_hosts force=true
    - name: add extra host
      shell: cat /tmp/extra_hosts >> /etc/hosts
      run_once: true

    - name: set host names
      shell: hostnamectl set-hostname {{ hostname }}

    - name: install docker
      yum: name=docker state=installed

EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./prepare.yaml
```

> 退出 master1 主机，执行命令

```
exit
```

Next:  [docker](/docker.md)[ ](/host-role.md)

