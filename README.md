# caas 产品部署手册

---

本文主要讲解caas产品的部署

## 简介

## 架构图

## Labs

本教程建议使用centos7 环境



# 实例

```
{

cat > admin-csr.json <<EOF


EOF
}
```

```
for instance in worker-0 worker-1 worker-2; do

done
```



```
cat <<EOF | sudo tee /bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

---

* 1
* 2

Next: [requirements](/requirements.md)

