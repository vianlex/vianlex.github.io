---
title: iptables 学习笔记
---
## iptables 简介
iptables 是 Linux 防火墙软件，虽然已经被 nftables 取代了，但是仍然广泛的运用着。iptables 防火墙的网络地址转换、数据包修改，以及过滤功能，是由 Linux 内核中的 netfilter 模块实现的，iptables 通过命令行去定义服务器流量的进出规则，然后由 netfilter 模块去执行。

## iptables Rule(规则)




## iptales 规则匹配说明
iptables 匹配规则是从上到下的一条一条的匹配的，某一条规则不管是匹配到 DROP 或者 ACCEPT 都算是匹配成功，一旦匹配成功就不会继续往下匹配了。




## 查看规则
查看 iptables 表规则时使用如下参数：
* -t 指定要查看的表，默认查看的是 filter 表
* -L 查看表的所有规则
* -n 表示不对 IP 地址进行反查，加上这个参数显示速度将会加快 
* -v 输出详细信息，包含通过该规则的数据包数量
* –line-number 显示规则的序列号，删除或修改规则时会用到
```
# 查看 filter 表的所有规则
iptables -nL 

# 查看 NAT 表的所有规则，并打印行号
iptables -t nat -nL --line-numbers

```
## 删除规则
删除整个表的规则命令 ` iptables [-t 表]  -F ` 常用例子如下：
```bash 
# 删除整个 filter 表的规则，不用 -t 指定表就默认时操作 filter 表
iptables -F 

# 删除 nat 表的整个规则
iptables -t nat -F
```
删除某一条规则的命令 `iptables -D 链 行号 `，例子如下：
```
# 先使用命令 iptables -t filter -nL 规则的链和行号，再删除
iptables -D INPUT 5 




```


