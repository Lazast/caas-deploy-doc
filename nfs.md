# nfs

---

nfs 安装

## 安装步骤

> 生成nfs配置文件

```
cat > config.py << EOF
role = {
    "identity": "{{ roles }}", # master or slave
    "port": "8080",
    "master_ip": "$CAAS_HOST_STORAGE1",
    "slave_ip": "$CAAS_HOST_STORAGE2",
}

EOF
```

> 安装nfs

```
cat > nfs.yaml << EOF

---
- hosts: storages
  tasks:
    - name: install the required yums for nfs
      yum:
        name: "{{ item }}"
        state: present
      with_items:
        - nfs-utils
        - rpcbind
        - python-devel
        - lvm2

    - name: create the directory /nfs
      file:
        path: /nfs
        state: directory

    - name: unarchive the nfsenv and convert2nfs to /opt/
      unarchive:
        src: "{{ item }}"
        dest: /opt/
      with_items:
        -
        -
- hosts: "{{ groups.storages[0] }}"
  vars:
    role: master
  tasks:
    template:
      src: ./config.py
      dest: /opt/convert2nfs/convert2nfs/controllers/config.py

- hosts: "{{ groups.storages[1] }}"
  vars:
    role: slave
  tasks:
    template:
      src: ./config.py
      dest: /opt/convert2nfs/convert2nfs/controllers/config.py

- hosts: storages
  tasks:
    - name: install the python interface for nfs
      shell: source /opt/nfsenv/bin/activate && cd /opt/conver2nfs && chmod +x bin/* &&  python setup.py install
    - name: start the nfs interface
      shell: source /opt/nfsenv/bin/activate && cd /opt/conver2nfs &&  uwsgi -d /var/log/convert2nfs.log --http-socket :8080 --venv /opt/nfsenv --pecan config.py

EOF
```

## 验证

Next: [haproxy,keepalived,rsync](/haproxykeepalivedrsync.md)

