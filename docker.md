# docker

---

## docker 配置

由于在上一篇文章中，我们已经通过ansible 自动全部安装完docker， 这里需要对docker 进行配置

> 在本地主机终端上，执行下面的命令， 获得所有主机列表

```
source ~/.bash_caas_env

hosts=$(env |grep CAAS_HOST_ |awk -F '=' '{print $2}')

echo $hosts
```

> 注意，如果您有iterm 可以使用iterm 的终端广播功能，如果没有，则执行下面的命令
>
> 下面的命令 会使用for 循环 顺序ssh 到每个host 上，直到 您输入 exit 命令，会退出当前的ssh 链接，进入下一个主机的ssh 链接， 这样我们就可以一直执行后续的docker 配置操作

```
for i in $hosts; do

ssh root@$i

done
```

> 接下来的每一次ssh 操作都会进入特定的主机，都要执行下面的命令

## 特定一台主机的docker 配置





## 验证

Next:  [keepalived & haproxy](/keepalivedandhaproxy.md)

