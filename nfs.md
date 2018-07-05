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



EOF
```

## 验证

Next: [openshift master](/openshift-master.md)

