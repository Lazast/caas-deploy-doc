# caas

---

caas portal等相关安装

## 安装步骤

> 安装配置mysql

```
cat > mysql.yaml << EOF

---
- hosts: storages
  tasks:
    - name: uninstall conflict mariadb-libs 
      yum: name=mariadb-libs state=absent
    - name: install depended package  
      yum: name=MySQL-python state=installed
    - name: install mysql-server 
      yum: name=MySQL-server state=installed
    - name: install mysql-client
      yum: name=mysql state=installed
    - name: mkdir mysql data dir 
      file: path=/caas_data/mysql_data/mysql state=directory
    - name: config mysql data dir
      shell: mysql_install_db --user=mysql --datadir=/caas_data/mysql_data/mysql
    - name: add iptable for mysql
      shell: iptables -I INPUT -p tcp --dport 3306 -j ACCEPT  && service iptables save

- hosts: "{{ groups.storages[0] }}"
  vars:
    root_password: "adminpwd"
    bind_address: "{{ groups.storages[0] }}"
  tasks:
    - name: copy mysql conf 
      template: src=../caas/mysql/my01.cnf  dest=/etc/my.cnf
    - name: start mysql
      service: name=mysql state=started enabled=false
    - name: MySQL | set root password
      mysql_user:
        name: "root"
        password: "{{ root_password }}" 
        check_implicit_admin: true
        login_user: "root"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present
    - name: MySQL | set replication slave
      mysql_user:
        name: "repl"
        login_user: "root"
        login_password: "{{ root_password }}"
        user: "repl"
        password: "slavepass"
        host: "{{ groups.storages[1] }}"
        priv: "*.*:replication slave:ALL"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present
    - name: MySQL | set replication slave
      mysql_user:
        name: "root"
        login_user: "root"
        login_password: "{{ root_password }}"
        user: "root"
        password: "{{ root_password }}"
        host: "{{ groups.storages[1] }}"
        priv: "*.*:ALL"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present


- hosts: "{{ groups.storages[1] }}"
  vars:
    root_password: "adminpwd"
    bind_address: "{{ groups.storages[1] }}"
  tasks:
    - name: copy mysql conf 
      template: src=../caas/mysql/my02.cnf  dest=/etc/my.cnf
    - name: start mysql
      service: name=mysql state=started enabled=false
    - name: MySQL | set root password
      mysql_user:
        name: "root"
        password: "{{ root_password }}" 
        check_implicit_admin: true
        login_user: "root"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present
    - name: MySQL | set replication slave
      mysql_user:
        name: "repl"
        login_user: "root"
        login_password: "{{ root_password }}"
        user: "repl"
        password: "slavepass"
        host: "{{ groups.storages[0] }}"
        priv: "*.*:replication slave:ALL"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present
    - name: MySQL | set replication slave
      mysql_user:
        name: "root"
        login_user: "root"
        login_password: "{{ root_password }}"
        user: "root"
        password: "{{ root_password }}"
        host: "{{ groups.storages[0] }}"
        priv: "*.*:ALL"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present

- hosts: "{{ groups.storages[0] }}"
  vars:
    root_password: "adminpwd"
  tasks:
    - name: get master file and position 
      mysql_replication:
        login_host: "{{ groups.storages[1] }}"
        login_user: "root"
        login_password: "{{ root_password }}" 
        mode: getmaster
      register: log_file_pos
    - name: MySQL | changea master status 
      command: "
        mysql -uroot -p{{ root_password }} -e
        \"
        STOP SLAVE;
        CHANGE MASTER TO MASTER_HOST='{{ groups.storages[1] }}', MASTER_USER='repl', MASTER_PASSWORD='slavepass', MASTER_LOG_FILE='{{ log_file_pos.File }}', MASTER_LOG_POS = {{ log_file_pos.Position }};
        START SLAVE;
        \"
        "

- hosts: "{{ groups.storages[1] }}"
  vars:
    root_password: "adminpwd"
  tasks:
    - name: get master file and position 
      mysql_replication:
        login_host: "{{ groups.storages[0] }}"
        login_user: "root"
        login_password: "{{ root_password }}" 
        mode: getmaster
      register: log_file_pos
    - name: MySQL | changea master status 
      command: "
        mysql -uroot -p{{ root_password }} -e
        \"
        STOP SLAVE;
        CHANGE MASTER TO MASTER_HOST='{{ groups.storages[0] }}', MASTER_USER='repl', MASTER_PASSWORD='slavepass', MASTER_LOG_FILE='{{ log_file_pos.File }}', MASTER_LOG_POS={{ log_file_pos.Position }};
        START SLAVE;
        \"
        "

- hosts: "{{ groups.storages[0] }}"
  vars:
    root_password: "adminpwd"
    caas_domain_os_console: "$CAAS_DOMAIN_OS_CONSOLE"
    caas_domain_portal_api: "$CAAS_DOMAIN_PORTAL_API"
    caas_vip_mysql_ldap: "$CAAS_VIP_MYSQL_LDAP"
    caas_domain_harbor: "$CAAS_DOMAIN_HARBOR"
    caas_vip_nfs: "$CAAS_VIP_NFS"
    caas_domain_prom: "$CAAS_DOMAIN_PROM"
    caas_domain_es: "$CAAS_DOMAIN_ES"
    caas_domain_pan: "$CAAS_DOMAIN_PAN"
  tasks:
    - name: MySQL | set root permission 
      mysql_user:
        name: "root"
        login_user: "root"
        login_password: "{{ root_password }}"
        user: "root"
        password: "{{ root_password }}"
        host: "%"
        priv: "*.*:ALL"
        login_unix_socket: "/caas_data/mysql_data/mysql/mysql.sock"
        state: present
    - name: copy hcpaas sql file 
      template: src=../caas/mysql/hcpaas_schema.sql dest=/caas_data/mysql_data/
    - name: copy hcpaas bmp sql file 
      template: src=../caas/mysql/hcpaas_bmp_schema.sql dest=/caas_data/mysql_data/

    - name: MySQL | init mysql db
      command: "
        mysql -uroot -p{{ root_password }} -e
        \"
        SOURCE /caas_data/mysql_data/hcpaas_schema.sql;
        SOURCE /caas_data/mysql_data/hcpaas_bmp_schema.sql;
        \"
        "
    - name: copy user clean script 
      template: src=../caas/mysql/user_clear.sh dest=/caas_data/mysql_data/ mode=0755
    - name: clear useless user 
      shell: /caas_data/mysql_data/user_clear.sh
EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./mysql.yaml
```

> 配置DNS

```
cat > domain.conf << EOF
address=/$CAAS_DOMAIN_LDAP/$CAAS_VIP_MYSQL_LDAP 
address=/$CAAS_DOMAIN_HARBOR/$CAAS_VIP_HARBOR 
address=/.$CAAS_DOMAIN_PAN/$CAAS_VIP_LOADBALANCE 
EOF


ansible -i ansible_hosts all -m copy -a 'src=./domain.conf dest=/etc/dnsmasq.d/'
ansible -i ansible_hosts all -m shell -a 'systemctl restart dnsmasq'
```

> 部署caasprotal

```
cat > caasportal.yaml << EOF
---
- hosts: localhost
  vars:
    caas_vip_mysql_ldap: $CAAS_VIP_MYSQL_LDAP
    harbor_url: $CAAS_DOMAIN_HARBOR
    omp_image_tag:  $CAAS_VAR_TAG_OMP_IMAGE
    agamaha_image_tag: $CAAS_VAR_TAG_AGAMAHA_IMAGE
    caas_vip_nfs: $CAAS_VIP_NFS
    caas_domain_os_console: $CAAS_DOMAIN_OS_CONSOLE
    haproxy_image_tag: $CAAS_VAR_TAG_HAPROXY_IMAGE
    hcenter_image_tag: $CAAS_VAR_TAG_HCENTER_IMAGE
    caas_domain_portal_api: $CAAS_DOMAIN_PORTAL_API
    muddle_image_tag: $CAAS_VAR_TAG_MUDDLE_IMAGE
    redis_image_tag: $CAAS_VAR_TAG_REDIS_IMAGE
    caas_domain_portal: $CAAS_DOMAIN_PORTAL

  tasks:
    - name: import the images for caasportal
      shell: cd ../images && ./import_caasportal.sh  $CAAS_DOMAIN_HARBOR Caas12345
    - name: create yaml file for caasportal
      template:
        src: ../caas/portal/ompall.yml
        dest: /tmp/ompall.yml
    - name: create project caasportal
      shell: oc login -uadmin -pCaas54321 && oc get project caasportal || oc new-project caasportal
    - name: set default scc to privileged and add privileged to project caasportal
      shell: ../caas/portal/addscc.sh
    - name: create secret deploycaas
      shell: oc get secret deploycaas || oc create secret docker-registry deploycaas -n caasportal --docker-username=admin --docker-password=Caas12345 --docker-email="admin@example.com" --docker-server="http://harbor.caas.example.com"
    - name: deploy the caasportal
      shell: oc login -uadmin -pCaas54321 && oc create -f /tmp/ompall.yml
EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./caasportal.yaml
```

> 部署promethues

```

```

## 验证

Next: [smoke test](/smoke-test.md)

