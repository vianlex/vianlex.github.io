---
title: iptables 学习笔记
---
## 1. iptables 简介
iptables 是 Linux 防火墙软件，虽然已经被 nftables 取代了，但是仍然广泛的运用着。iptables 防火墙的网络地址转换、数据包修改，以及过滤功能，是由 Linux 内核中的 netfilter 模块实现的，iptables 通过命令行去定义服务器流量的进出规则，然后由 netfilter 模块去执行。

## 2. iptables 过滤和转发数据包的流程
流量数据流量包传输到主机的数据链路层时，也就传输到内核的 TCP/IP 协议栈时，会经过 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING 链路节点，通过 iptable 可以在这些节点上配置过滤、转发规则，每一个节点上可以配置多个规则，多个规则串联起来形成一条链(chain)，在节点中的链规则是从上往下执行的，只要某一条规则匹配成功，就不继续往下匹配，并执行规则中配置的 ACCEPT 或者 DROP 等等动作，链中的规则，按规则作用分组，然后每一个分组可以看作一个表(table)，因此不同作用规则要定义到指定的表中，表的说明如下：
- raw 表：控制 nat 表中连接追踪机制的启用状况，可以控制的链路节点有 PREROUTING、OUTPUT;
- mangle 表：用于修改数据包中的数据，可以控制的链路节点有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING;
- nat 表：用于数据包的转发，可以控制的链路节点有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING;
- filter 表：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT。
 **总上所述，iptable 中常说的4表5链，指的就是 raw、mangle、nat、filter 和 PREROUTING、INPUT、OUTPUT、FORWARD、POSTROUTING。**
![iptables-过滤转发流程](/images/iptables-过滤转发流程.png)

## iptales 规则
netfilter 是从上到下的一条一条的匹配 iptables 在 raw -> mangle -> nat -> filter 表中定义的规则，只要匹配成功就会执行规则中配置的动作，注意一旦匹配成功就不会继续往下匹配了。
### 2.1 iptables 规则定义
iptables 定义规则的语法如下：
```
iptables -t <table> <command> <chain> <parameter> -j <target>

```
* table 指定规则定义在哪个表中，如 raw、mangle、nat、filter 等。
* command 指定规则的添加方式，如 -A 将规则追加到表的末尾，-I 将规则插入到表的指定行中。
* chain 指定规则要定义在哪个节点链上
* parameter 指定规则的过滤或者转发的条件参数
* target 指定规则匹配要跳转执行的动作，如 ACCEPT、DROP 等等

### 2.2 定义规则用到的参数说明
定义规则常用的参数说明如下:


### 2.3 定义规则用到的动作说明
- ACCEPT：允许数据包通过。
- DROP：直接丢弃数据包，不给任何回应信息。
- REJECT：拒绝数据包通过，会响应一个拒绝的信息。
- SNAT：源地址转换，解决内网用户用同一个公网地址上网的问题。
- MASQUERADE：是 SNAT 的一种特殊形式，适用于动态的、临时会变的 ip 上。
- DNAT：目标地址转换。
- REDIRECT：在本机做端口映射。
- LOG：会将访问日志记录到 /var/log/messages 文件中，只是做日记记录，所以会继续往下匹配规则。






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
查询规则显示的结果说明
- pkts 规则匹配到的报文的个数。
- bytes 匹配到的报文包的大小总和。
- target 规则执行的动作。
- prot 表示规则作用的协议。
- opt 表示规则对应的选项。
- in 表示数据包由哪个接口(网卡)流入，定义规则的时候，可以设置。
- out 表示数据包由哪个接口(网卡)流出，定义规则的时候，可以设置。
- source 表示规则对应的源头地址，可以是一个IP，也可以是一个网段。
- destination 表示规则对应的目标地址，可以是一个 IP，也可以是一个网段。
当然，我们也可以只查看某个链的规则，并且不让IP进行反解，这样更清晰一些，比如 iptables -nvL INPUT
- line-numbers 显示规则在对应表中的行号，删除的时候，指定行号去删除。
- policy 表示当前链的默认策略，表示所有规则都没有匹配时，默认指定的动作。
- packets 表示当前链（上例为INPUT链）默认策略匹配到的包的数量，0 packets表示默认策略匹配到0个包。
- bytes 表示当前链默认策略匹配到的所有包的大小总和。





## 删除规则
删除整个表的规则命令 ` iptables [-t 表]  -F ` 常用例子如下：
```bash 
# 删除整个 filter 表的规则，不用 -t 指定表就默认时操作 filter 表
iptables -F 

# 删除 nat 表的整个规则
iptables -t nat -F
```
删除某一条规则的命令 `iptables - <table> -D <chain> <line-number> `，例子如下：
```
# 查看规则的 chain 和 line-number
iptables -t filter -nL
# 删除规则
iptables -t filter -D INPUT 5  

```

