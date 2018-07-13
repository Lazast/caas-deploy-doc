# Prerequistes

---

安装caas 平台提前需要准备的工具

## 免密登录

> 在CAAS\_HOST\_MASTER1上执行如下命令,对所有主机免密登录

```bash
hosts=$(env |grep CAAS_HOST_ |awk -F '=' '{print $2}')

if [ ! -f ~/.ssh/id_rsa.pub ]; then
    ssh-keygen -t rsa -b 1024 -C "root"
fi
```

> copy 秘钥

```
for h in $hosts; do
    ssh-copy-id root@$h
done
```

## 安装离线包

### 配置离线安装包

> 在CAAS\_HOST\_MASTER1 上 ，找一块分区，分区大小必须 &gt; 50G， 若没有大于50G的分区，请联系客户或相关人员，增加盘或者划分分区
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
else
   echo "找到>50G的分区目录: $validdata"
   offlinedata=$validdata
   echo "export offlinedata=$validdata" >> ~/.bashrc
   mkdir -p $offlinedata
fi
```

> 将caas-offline.tar离线包上传到到 CAAS\_MASTER1 机器的目录$offlinedata下

解压离线文件

```bash
tar xvf $offlinedata/caas-offline.tar -C $offlinedata

cd $offlinedata/caas-offline/cent7.2

# 启动 http 服务，搭建caas的yum源

nohup python -m SimpleHTTPServer 38888 &

# 配置iptables 规则，其实能访问
iptables -I INPUT -p tcp  --dport 38888 -j ACCEPT  && service iptables save
```

## master1配置ansible

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
harbor_host="`env|grep CAAS_VIP_HARBOR|awk -F= '{print $2}'` `env|grep CAAS_DOMAIN_HARBOR |awk -F= '{print $2}'`"
ldap_host="`env|grep CAAS_VIP_MYSQL_LDAP|awk -F= '{print $2}'` `env|grep CAAS_DOMAIN_LDAP |awk -F= '{print $2}'`"
grep "$harbor_host" ./extra_hosts || echo $harbor_host >> ./extra_hosts
grep "$ldap_host" ./extra_hosts || echo $ldap_host >> ./extra_hosts
echo "$CAAS_VIP_LOADBALANCE $CAAS_DOMAIN_OS_CONSOLE" >> ./extra_hosts
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

> 检查所有master和node节点的selinux, **当下面命令运行出现错误时，请手动重启所有master和node节点, 之后再次运行该命令，确保该修改成功**

```
cat > selinux-check.yaml << EOF

---
- hosts: masters,nodes
  tasks:
    - name: enable selinux Persist
      shell: echo "SELINUX=enforcing" > /etc/selinux/config
    - name: enable selinux Persist
      shell: echo "SELINUXTYPE=targeted" >> /etc/selinux/config
    - name: check selinux
      shell: setenforce 1
      register: result
      failed_when: "result.rc != 0 or 'SELinux is disabled' in result.stderr"

EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./selinux-check.yaml
```

**若上边操作重启了master1节点，请在master1上执行下面“开始”-“结束”之间的命令，否则不必执行**

\#\#\#\#\#\#开始\#\#\#\#\#

```
cd $offlinedata/caas-offline/cent7.2

# 重新启动 http 服务，搭建caas的yum源

nohup python -m SimpleHTTPServer 38888 &

# 进入caas安装目录
cd $offlinedata/caas-offline/install
```

\#\#\#\#\#\#结束\#\#\#\#\#

> 生成prepare配置文件， 进行环境的一些准备工作

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

    - name: base packages install
      yum: 
        name: "{{ item }}"
        state: present
      with_items:
        - wget 
        - git 
        - net-tools 
        - bind-utils 
        - yum-utils 
        - iptables-services 
        - bridge-utils 
        - bash-completion 
        - kexec-tools 
        - sos 
        - psacct
        - PyYAML
        - python-ipaddress

    - name: copy caas host resolve to all hosts
      copy: src=./extra_hosts dest=/tmp/extra_hosts force=true
    - name: add extra host
      shell: cat /tmp/extra_hosts >> /etc/hosts

    - name: set host names
      shell: hostnamectl set-hostname {{ hostname }}

    - name: disable firewalld
      service: name=firewalld state=stopped enabled=no

- hosts: storages
  tasks:
    - name: Disable selinux
      shell: setenforce 0
      register: result
      failed_when: "result.rc != 0 and 'SELinux is disabled' not in result.stderr"
    - name: Disable selinux Persist
      shell: echo "SELINUX=disabled" > /etc/selinux/config
    - name: Disable selinux Persist
      shell: echo "SELINUXTYPE=targeted" >> /etc/selinux/config

- hosts: masters 
  tasks:
    - name: install java 
      yum: name=java state=present

EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./prepare.yaml
```

Next:  [docker](/docker.md)[ ](/host-role.md)

