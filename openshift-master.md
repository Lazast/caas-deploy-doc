# openshift

---

安装openshift

## 安装步骤

> 配置LB节点keepalived，生成配置文件

```
cat > keepalived.conf << EOF

! Configuration File for keepalived
vrrp_instance VI_1 {
    state {{ lb_role }}
    interface eth0
    virtual_router_id 85
    priority {{ pri }}
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        $CAAS_VIP_LOADBALANCE
    }
}
EOF

```

> 配置LB节点keepalived

```
cat > keepalived.yaml << EOF

---
- hosts: "{{ groups.lbs[0] }}"
  vars:
    lb_role: "MASTER"
    pri: 101
  tasks:
    - name: install keepalived 
      yum: name=keepalived state=present
    - name: copy keeplived config
      template: src=./keepalived.conf dest=/etc/keepalived/keepalived.conf
    - name: start keepalived 
      service: name=keepalived state=restarted enabled=true

- hosts: "{{ groups.lbs[1] }}"
  vars:
    lb_role: "BACKUP"
    pri: 99 
  tasks:
    - name: install keepalived 
      yum: name=keepalived state=present
    - name: copy keeplived config
      template: src=./keepalived.conf dest=/etc/keepalived/keepalived.conf
    - name: start keepalived 
      service: name=keepalived state=restarted enabled=true

EOF
```

> 生成openshift inventory变量配置

```
cat > ansible_os_hosts << EOF

[OSEv3:children]
masters
nodes
etcd
lb

[OSEv3:vars]
ansible_ssh_user=root
#ansible_ssh_port=3222
# 日志输出级别 
# 0:错误和警告
# 2:默认级别
# 4:调式级别
# 6:API调式级别
# 8:API包体调式级别
debug_level=2
containerized=false
openshift_is_containerized=false
openshift_deployment_type=origin
#enable_excluders=false
# 多租户网络
os_sdn_network_plugin_name=redhat/openshift-ovs-multitenant

openshift_master_identity_providers=[{'name': 'my_ldap_provider', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider', 'attributes': {'id': ['dn'], 'email': ['mail'], 'name': ['cn'], 'preferredUsername': ['cn']}, 'bindDN': 'cn=admin,dc=general,dc=com', 'bindPassword': 'generalcom', 'ca': '', 'insecure': 'true', 'url': 'ldap://$CAAS_DOMAIN_LDAP:389/ou=users,dc=general,dc=com?cn'}]

openshift_install_examples=true
openshift_clock_enabled=true
openshift_check_min_host_disk_gb=1
# 安装使用的版本
openshift_pkg_version=-3.9.0
openshift_version=3.9.0
openshift_image_tag=v3.9.0
openshift_master_cluster_method=native
openshift_master_cluster_hostname=$CAAS_DOMAIN_OS_CONSOLE
openshift_master_cluster_public_hostname=$CAAS_DOMAIN_OS_CONSOLE
openshift_master_default_subdomain=$CAAS_DOMAIN_PAN
openshift_router_selector='region=master'
openshift_registry_selector='region=master'

# Configure master API and console ports.
openshift_master_api_port=8443
openshift_master_console_port=8443

# logging configuration
openshift_logging_image_version=v3.9
openshift_logging_image_prefix=$CAAS_DOMAIN_HARBOR/openshift/origin-
openshift_logging_use_ops=true
openshift_logging_master_public_url=https://$CAAS_DOMAIN_OS_CONSOLE:8443
openshift_logging_install_logging=true
openshift_logging_es_allow_external=true
openshift_logging_curator_default_days=14

#prometheus configuration
#openshift_hosted_prometheus_deploy=true

#Disable origin-service-catalog
openshift_enable_service_catalog=false

# for docker
docker_upgrade=False
openshift_docker_options="--log-driver=json-file --log-opt max-size=50m -l warn --ipv6=false"
openshift_docker_insecure_registries=$CAAS_DOMAIN_HARBOR

oreg_url=$CAAS_DOMAIN_HARBOR/openshift/origin-\${component}:\${version}
#openshift_examples_modify_imagestreams=true
# 在下载镜像之前，首先选择addition_registry下载镜像
openshift_docker_additional_registries=$CAAS_DOMAIN_HARBOR

# 系统中主机ip的掩码为9位，系统支持的最大主机数为2^9=512
osm_host_subnet_length=$CAAS_OSM_HOST_SUBNET_LENGTH
# 每个主机上的pod ip掩码位数为18-9=9，即每台主机上最多有2^9=512个pod的ip
osm_cluster_network_cidr=$CAAS_OSM_CLUSTER_NETWORK_CIDR

# Configure node kubelet arguments. pods-per-core is valid in OpenShift Origin 1.3 or OpenShift Container Platform 3.3 and later.
openshift_node_kubelet_args={'pods-per-core': ['10'], 'max-pods': ['320'], 'image-gc-high-threshold': ['90'], 'image-gc-low-threshold': ['80']}

openshift_master_identity_providers=[{'name': 'my_ldap_provider', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider', 'attributes': {'id': ['dn'], 'email': ['mail'], 'name': ['cn'], 'preferredUsername': ['cn']}, 'bindDN': 'cn=admin,dc=general,dc=com', 'bindPassword': 'generalcom', 'ca': '', 'insecure': 'true', 'url': 'ldap://$CAAS_DOMAIN_LDAP:389/ou=users,dc=general,dc=com?cn'}]

# for ca
openshift_ca_cert_expire_days=1825
openshift_node_cert_expire_days=1825
openshift_master_cert_expire_days=1825
etcd_ca_default_days=1825
#for webconsole
openshift_web_console_prefix=$CAAS_DOMAIN_HARBOR/openshift/
openshift_web_console_image_name=origin-web-console
openshift_web_console_version=v3.9


EOF
```

> 生成openshift inventory主机配置，并安装openshift

```
env |grep CAAS_HOST_MASTER |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/masters
env |grep CAAS_HOST_NODE |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/nodes
env |grep CAAS_HOST_LB |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/lbs
env |grep CAAS_HOST_STORAGE |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print $2" hostname="arrays[3]}}' > /tmp/storage

cat >> ansible_os_hosts <<EOF

[masters]
$(cat /tmp/masters)
[etcd]
$(cat /tmp/masters)
[lb]
$(cat /tmp/lbs)
[nodes]
$(cat /tmp/masters | awk '{print $1 " openshift_node_labels=\"{'\''region'\'': '\''master'\''}\" openshift_schedulable=true"}')
$(cat /tmp/nodes | awk '{print $1 " openshift_node_labels=\"{'\''region'\'': '\''node'\''}\" openshift_schedulable=true"}')

EOF


# import image for openshift
yum install -y unzip
unzip -q ../images/os39-base-images.zip -d ../images/
cd ../images/os39-base-images/
./import.sh $CAAS_DOMAIN_HARBOR Caas12345
cd -

tar -xvf ../openshift/openshift-ansible.tar -C ../openshift/

ansible-playbook -i ./ansible_os_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ../openshift/openshift-ansible/playbooks/deploy_cluster.yml
# 当前节点为master1
oc adm policy add-cluster-role-to-user cluster-admin admin
```
> 添加loadbalance节点80,443,30000-32767端口部分
```
cat > config-haproxy-lb << EOF
# Global settings
#---------------------------------------------------------------------
global
    maxconn     20000
    log         /dev/log local0 info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
#    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          300s
    timeout server          300s
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 20000

listen stats
    bind :9000
    mode http
    stats enable
    stats uri /

frontend  atomic-openshift-api
    bind *:8443
    default_backend atomic-openshift-api
    mode tcp
    option tcplog

backend atomic-openshift-api
    balance source
    mode tcp
    server      master0 {{ groups.masters[0] }}:8443 check
    server      master1 {{ groups.masters[1] }}:8443 check
    server      master2 {{ groups.masters[2] }}:8443 check

frontend  atomic-openshift-http
    bind *:80
    default_backend atomic-openshift-http
    mode tcp
    option tcplog

backend atomic-openshift-http
    balance source
    mode tcp
    server      master0 {{ groups.masters[0] }}:80 check
    server      master1 {{ groups.masters[1] }}:80 check
    server      master2 {{ groups.masters[2] }}:80 check

frontend  atomic-openshift-https
    bind *:443
    default_backend atomic-openshift-https
    mode tcp
    option tcplog

backend atomic-openshift-https
    balance source
    mode tcp
    server      master0 {{ groups.masters[0] }}:443 check
    server      master1 {{ groups.masters[1] }}:443 check
    server      master2 {{ groups.masters[2] }}:443 check

frontend tcp_30000-32767
    bind *:30000-32767
    mode tcp
    default_backend tcp_30000-32767

backend tcp_30000-32767
    balance source
    mode tcp
    server      master0 {{ groups.masters[0] }}
    server      master1 {{ groups.masters[1] }}
    server      master2 {{ groups.masters[2] }}
EOF

cat > config-haproxy-lb.yml << EOF
- hosts: lb
  tasks:
    - name: generate the configuration file for haproxy of lb
      template:
        src: config-haproxy-lb
        dest: /etc/haproxy/haproxy.cfg
    - name: restart haproxy to make the configuration take effect
      systemd:
        name: haproxy
        state: restarted
EOF

ansible-playbook -i ./ansible_os_hosts --ssh-common-args "-o StrictHostKeyChecking=no" config-haproxy-lb.yml
```


## 验证

Next:[caas ](/caas.md)

