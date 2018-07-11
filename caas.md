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

EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./mysql.yaml
```

## 

## 验证

Next: [smoke test](/smoke-test.md)

