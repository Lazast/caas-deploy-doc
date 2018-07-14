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
    - name: 
    - name: create secret deploycaas
      shell: oc get secret deploycaas || oc create secret docker-registry deploycaas -n caasportal --docker-username=admin --docker-password=Caas12345 --docker-email="admin@example.com" --docker-server="http://$CAAS_DOMAIN_HARBOR"
    - name: deploy the caasportal
      shell: oc login -uadmin -pCaas54321 && oc create -f /tmp/ompall.yml
EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./caasportal.yaml
```

> 部署promethues

```
mkdir -p prometheus
cp ../caas/prometheus/config/alert.rules prometheus/
cp ../caas/prometheus/config/config.yml prometheus/

cat > prometheus/prometheus-pods.yml << EOF
---
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: prometheus
  namespace: prometheus
  labels:
    name: prometheus-deployment
spec:
  replicas: 1
  selector:
    app: prometheus
  template:
    metadata:
      namespace: prometheus
      labels:
        app: prometheus
    spec:
      volumes:
        - name: data
          hostPath:
            path: /prometheus
        - name: config-volume
          configMap:
            name: prometheus-config
        - name: rule
          persistentVolumeClaim:
            claimName: promrule
      containers:
        - name: prometheus
          image: $CAAS_DOMAIN_HARBOR/openshift/prometheus:v1.8.1
          command:
            - /bin/prometheus
          args:
            - '-config.file=/etc/prometheus/prometheus.yml'
            - '-storage.local.path=/prometheus'
            - '-storage.local.retention=168h'
            - '-alertmanager.url=http://alertmanager:9093'
            - '-web.external-url=http://$CAAS_DOMAIN_PROM'
          ports:
            - containerPort: 9090
              protocol: TCP
          resources:
            limits:
              cpu: '8'
              memory: 4Gi
            requests:
              cpu: '1'
              memory: 2Gi
          volumeMounts:
            - name: data
              mountPath: /prometheus
            - name: config-volume
              mountPath: /etc/prometheus
            - name: rule
              mountPath: /rules
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: true
      restartPolicy: Always
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/hostname: $(env |grep CAAS_HOST_NODE1 |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print arrays[3]}}') 

---
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: alertmanager
  namespace: prometheus
  labels:
    name: alertmanager
spec:
  replicas: 1
  selector:
    app: alertmanager
  template:
    metadata:
      labels:
        app: alertmanager
    spec:
      volumes:
        - name: config
          configMap:
            name: alertmanager
        - name: data
          persistentVolumeClaim:
            claimName: alertmanager
      containers:
        - name: alertmanager
          image: $CAAS_DOMAIN_HARBOR/openshift/alertmanager
          command:
            - /bin/alertmanager
          args:
            - '-config.file=/etc/alertmanager/config.yml'
            - '-storage.path=/alertmanager'
            - '-web.external-url=http://$CAAS_DOMAIN_ALERT/'
          ports:
            - containerPort: 9093
              protocol: TCP
          resources:
            limits:
              cpu: '1'
              memory: 2500Mi
            requests:
              cpu: 100m
              memory: 100Mi
          volumeMounts:
            - name: data
              mountPath: /alertmanager
            - name: config
              mountPath: /etc/alertmanager
          terminationMessagePath: /dev/termination-log
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/hostname: $(env |grep CAAS_HOST_NODE1 |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print arrays[3]}}')
---
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: grafana
  namespace: prometheus 
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    app: grafana
    deploymentconfig: grafana
  template:
    metadata:
      labels:
        app: grafana
        deploymentconfig: grafana
    spec:
      volumes:
        - name: grafana-2
          emptyDir: {}
        - name: grafana-3
          emptyDir: {}
      containers:
        - name: grafana
          image: $CAAS_DOMAIN_HARBOR/openshift/grafana
          ports:
            - containerPort: 3000
              protocol: TCP
          env:
            - name: GF_SECURITY_ADMIN_PASSWORD
              value: grafana_1234
          resources: {}
          volumeMounts:
            - name: grafana-2
              mountPath: /var/lib/grafana
            - name: grafana-3
              mountPath: /var/log/grafana
          terminationMessagePath: /dev/termination-log
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}

---
apiVersion: v1
kind: Service
metadata:
  name: alertmanager
  namespace: prometheus
  labels:
    app: alertmanager
spec:
  ports:
    - name: 9093-tcp
      protocol: TCP
      port: 9093
      targetPort: 9093
  selector:
    deploymentconfig: alertmanager
  type: NodePort

---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: prometheus
  labels:
    app: prometheus
spec:
  ports:
    - name: 9090-tcp
      protocol: TCP
      port: 9090
      targetPort: 9090
  selector:
    deploymentconfig: prometheus
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: prometheus
  labels:
    app: grafana
spec:
  ports:
    - name: 3000-tcp
      protocol: TCP
      port: 3000
      targetPort: 3000
  selector:
    deploymentconfig: grafana
  type: ClusterIP

#Follows are the routes 
---
apiVersion: v1
kind: Route
metadata:
  name: grafana
  namespace: prometheus
spec:
  host: $CAAS_DOMAIN_GRAFANA 
  to:
    kind: Service
    name: grafana
  port:
    targetPort: 3000-tcp
  wildcardPolicy: None
---
apiVersion: v1
kind: Route
metadata:
  name: prometheus
  namespace: prometheus
spec:
  host: $CAAS_DOMAIN_PROM 
  to:
    kind: Service
    name: prometheus
    weight: 100
  port:
    targetPort: 9090-tcp
  wildcardPolicy: None
---
apiVersion: v1
kind: Route
metadata:
  name: alertmanager
  namespace: prometheus
spec:
  host: $CAAS_DOMAIN_ALERT 
  to:
    kind: Service
    name: alertmanager
    weight: 100
  port:
    targetPort: 9093-tcp
  wildcardPolicy: None
EOF


cat > prometheus/volumes.yml << EOF
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: alertmanager
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: "1024Mi"
  nfs:
    path: /nfs/alertmanager/
    server: $CAAS_VIP_NFS
  persistentVolumeReclaimPolicy: Retain

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: promrule
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: "2048Mi"
  nfs:
    path: /nfs/prometheusrule
    server: $CAAS_VIP_NFS
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alertmanager
  namespace: prometheus
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1024Mi
  volumeName: alertmanager

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: promrule
  namespace: prometheus
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 2048Mi
  volumeName: promrule
EOF

cat ../caas/prometheus/config/prometheus.yaml > prometheus/prometheus.yml
hostname_ip=$(env |grep CAAS_HOST |awk -F '=' '{if ($2!="") { split(tolower($1),arrays, "_"); print arrays[3]" " $2}}')
tempfile=$(mktemp temp.XXXXXX)
index=0
host=""
ip=""
for i in $hostname_ip;do
    let tmp=$index%2  
    if [ $tmp -ne 0 ];then
        ip=$i
cat >> $tempfile << EOF
    - targets: ['$ip:9100']
      labels:
        alias: $host 
EOF
    else
        host=$i
    fi
    let index=$index+1
done

cat $tempfile >> prometheus/prometheus.yml


cat > prometheus-setup.sh << EOF
curl -X GET 'http://$CAAS_VIP_NFS:8080/nfs/create?volName=prometheusrule&volSize=2048'
curl -X GET 'http://$CAAS_VIP_NFS:8080/nfs/create?volName=alertmanager&volSize=1024'

oc login --username=admin --password=Caas54321 --insecure-skip-tls-verify=true 
oc project prometheus  || oc new-project prometheus

oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:prometheus:default
oc adm policy add-scc-to-user privileged system:serviceaccount:prometheus:default
oc create configmap prometheus-config --from-file=prometheus/prometheus.yml
oc create configmap alertmanager --from-file=prometheus/config.yml


oc create -f prometheus/volumes.yml
sleep 5
oc create -f prometheus/prometheus-pods.yml

scp prometheus/alert.rules $CAAS_VIP_NFS:/nfs/prometheusrule/

EOF

cat > node_exporter.yml << EOF
---
- hosts: all
  tasks: 
    - name: copy the node_exporter to all the server
      copy:
        src: ../caas/prometheus/node_exporter
        dest: /usr/bin/node_exporter
        mode: 0755
    - name: start the node_exporter
      shell: 'nohup /usr/bin/node_exporter &'
    - name: open the port:9100 for node_exporter
      shell: 'iptables -nL|grep 9100 || iptables -I INPUT -p tcp --dport 9100 -j ACCEPT && iptables-save'
EOF

ansible-playbook -i ansible_hosts node_exporter.yml

chmod +x  prometheus-setup.sh
./prometheus-setup.sh

```

## 验证

Next: [smoke test](/smoke-test.md)

