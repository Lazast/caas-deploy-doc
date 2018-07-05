# docker

---

## docker 配置

由于在上一篇文章中，我们已经通过ansible 自动全部安装完docker， 这里需要对docker 进行配置

> 在本地主机终端上，执行下面的命令， 获得所有主机列表

```
source ~/.bash_caas_env

env |grep CAAS_HOST_ |awk -F '=' '{print $2}'

```

> 注意，如果您有iterm 可以使用iterm 的终端广播功能
>
> 接下来 要顺序ssh登陆所有的打印出来的主机 ,都要执行后续的命令

## 特定一台主机的docker 配置

## 验证

Next:  [keepalived & haproxy](/keepalivedandhaproxy.md)

