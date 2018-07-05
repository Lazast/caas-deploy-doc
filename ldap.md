# ldap

---

ldap 安装

## 安装步骤

```
cat > ldap.yaml << EOF

---
- hosts: storages
  tasks:
    - name: copy ldap image
      copy: src=../images/ldap/ dest=/tmp/ force=true
    - name: load ldap image 
      shell: docker load < /tmp/ldap.tar
    - name: install ldap 
      yum: name=openldap-clients state=installed

- hosts: "{{ groups.storages[0] }}"
  tasks:
    - name: install ldap
      shell: docker run --name ldap1 --add-host ldap2.general.com:{{ groups.storages[1] }}  --restart=always  --hostname ldap1.general.com --env LDAP_BASE_DN="dc=general,dc=com" --env LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://ldap1.general.com:389','ldap://ldap2.general.com:3389']" --env LDAP_REPLICATION=true --env LDAP_TLS_VERIFY_CLIENT="never" --env LDAP_DOMAIN="general.com" --env LDAP_ADMIN_PASSWORD="generalcom" -v /caas_data/ldap_data/ldap_config:/etc/ldap/slapd.d -v /caas_data/ldap_data/ldap_data:/var/lib/ldap -v /etc/localtime:/etc/localtime:ro -p 3389:389 --detach osixia/openldap:1.2.1 --loglevel debug --copy-service

- hosts: "{{ groups.storages[1] }}"
  tasks:
    - name: install ldap
      shell: docker run --name ldap2 --add-host ldap1.general.com:{{ groups.storages[0] }}  --restart=always  --hostname ldap2.general.com --env LDAP_BASE_DN="dc=general,dc=com" --env LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://ldap1.general.com:3389','ldap://ldap2.general.com:389']" --env LDAP_REPLICATION=true --env LDAP_TLS_VERIFY_CLIENT="never" --env LDAP_DOMAIN="general.com" --env LDAP_ADMIN_PASSWORD="generalcom" -v /caas_data/ldap_data/ldap_config:/etc/ldap/slapd.d -v /caas_data/ldap_data/ldap_data:/var/lib/ldap -v /etc/localtime:/etc/localtime:ro -p 3389:389 --detach osixia/openldap:1.2.1 --loglevel debug --copy-service

- hosts: "{{ groups.storages[0] }}"
  tasks:
    - name: ldap config 
      shell: sleep 3 && ldapadd -x -h 127.0.0.1 -p 3389 -D cn=admin,dc=general,dc=com -w generalcom -f /tmp/user.ldif


- hosts: "{{ groups.storages[1] }}"
  tasks:
    - name: ldap config 
      shell: ldapsearch -x -h 127.0.0.1 -p 3389 -b dc=general,dc=com  -D cn=admin,dc=general,dc=com -w generalcom

EOF

ansible-playbook -i ./ansible_hosts --ssh-common-args "-o StrictHostKeyChecking=no" ./ldap.yaml
```

## 验证

Next: [harbor](/harbor.md)

