# requirements

---

硬件需求

## hardware requirements


| 设备类型 | 用途 | 配置 | 数量 |
| :--- | :--- | :--- | :--- |
| 服务器 | 存储节点 |1.  至少2路Intel E5-2630 v4 以上处理器  2.  至少64G内存  3.  至少6T SATA\*4磁盘 ，品牌特性均不限制  4.  2个万兆网卡接口；2个千兆网卡接口\(配置远程管理卡，远程管理卡支持IPMI\) | 2 |
| 服务器 | master节点 | 1.  至少2路Intel E5-2630 v4 以上处理器  2.  至少32G内存  3.  至少2.4TSATA\*2磁盘 ，品牌特性均不限制 4.  2个万兆网卡接口；2个千兆网卡接口\(配置远程管理卡，远程管理卡支持IPMI\) | 3 |
| 服务器 | node节点 | 1. 至少2路Intel E5-2630 v4 以上处理器  2.  至少64G内存  3.  6TSATA\*2 ，品牌特性均不限制  4.  2个万兆网卡接口；2个千兆网卡接口\(配置远程管理卡，远程管理卡支持IPMI\)  | 适需求而定，建议最少两台 |
| 服务器 | loadbalance节点 | 1.  至少2路Intel E5-2630 v4 以上处理器  2.  至少32G内存  3.  300GSATA\*2 ，品牌特性均不限制  4.  1个万兆网卡接口；2个千兆网卡接口\(配置远程管理卡，远程管理卡支持IPMI\) | 2 |


## software requirements
|  | 预先定义的内容 | 备注 |
| :--- | :--- | :--- |
| ldap | ldap管理员账号：cn=admin,dc=general,dc=com {password}；   os管理员账号：cn=admin,ou=users,dc=general,dc=com {password} | ldap管理员账号在部署ldap时通过LDAP_ADMIN_PASSWORD设置； os管理员账号在部署完之后在创建的user.ldif中设置|
| harbor | harbor域名：{harbor.example.com}；harbor管理员（账号默认admin）密码：{password}  {caasportal api域名} | 在部署harbor的时候设置密码、域名以及caasportal apiserver的域名 |
| mysql | mysql 用户名，密码（密码需设置为adminpwd） | 用于caasportal，不设置为adminpwd的话，则需要在部署caasporta的muddle的时候设置相关的环境变量 |
| NFS | 需要设置nfs接口程序的端口 | 在运行nfs接口程序的命令中指定 |
| openshift | 设置web console需要访问的域名；设置ldap中的管理员账号、密码；设置默认生成的的url的子域名 |  |
| prometheus | 设置alertmanager、prometheus、grafana的域名；grafana的管理员密码 | 建议alertmanager.{openshift中设置的子域名}；prometheus.{openshift中设置的子域名}；grafana.{openshift中设置的子域名} |
| elasticsearh | es的域名为自动生成，格式为es.{openshift中设置的子域名} |  |
| caasportal| caasportal的域名；caasportal api的域名 | 建议 portal.{openshift中设置的子域名}；portalapi.{openshift中设置的子域名} |
| 泛域名 | 用于CaaS平台所有容器service默认的解析域名 | 例如：*.caas.domain.com做泛域名解析到VIP1 |
| VIP | VIP1 用于openshift loadbalance 上的haproxy负载；VIP2 用于存储机上的haproxy代理ldap、MySQL负载；VIP3 用于存储机上harbor的高可用；VIP4 用于存储机上NFS的高可用 |  |

centos 7.2

Next: [host roles](https://legacy.gitbook.com/book/jiulongzaitian/caas/edit#)

