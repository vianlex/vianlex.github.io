---
title: iptables 学习笔记
---





## 查看规则
查看 iptables 表规则时使用如下参数：
* -t 指定要查看的表，默认查看的是 filter 表
* -L 查看表的所有规则
* -n 输出详细信息，包含通过该规则的数据包数量
* -v 跟 -n 一样输出详细信息，只是显示方式不一样
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


