---
title: iptables 学习笔记
---
## 1. iptables 简介
iptables 是 Linux 防火墙软件，虽然已经被 nftables 取代了，但是仍然广泛的运用着。iptables 防火墙的网络地址转换、数据包修改，以及过滤功能，是由 Linux 内核中的 netfilter 模块实现的，iptables 通过命令行去定义服务器流量的进出规则，然后由 netfilter 模块去执行。

## 2. iptables 过滤和转发数据包的流程
流量数据流量包传输到主机的数据链路层时，也就传输到内核的 TCP/IP 协议栈时，会经过 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING 链路节点，通过 iptable 可以在这些节点上配置过滤、转发规则，每一个节点上可以配置多个规则，多个规则串联起来形成一条链(chain)，在节点中的链规则是从上往下执行的，只要某一条规则匹配成功，就不继续往下匹配，并执行规则中配置的 ACCEPT 或者 DROP 等等动作，链中的规则，按规则作用分组，然后每一个分组可以看作一个表(table)，在表中的规则作用如下：
- raw：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT;
- mangle 表：用于修改数据包中的数据，可以控制的链路有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING;
- nat 表：用于数据包的转发，可以控制的链路有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING;
- filter 表：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT。
![iptables-过滤转发流程](/images/iptables-过滤转发流程.png)




## iptales 规则说明
iptables 匹配规则是从上到下的一条一条的匹配的，只要匹配成功就会执行规则中配置的动作，注意一旦匹配成功就不会继续往下匹配了。







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


