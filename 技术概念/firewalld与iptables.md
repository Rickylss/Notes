# iptables概念详解

> iptables防火墙，主要实现封包过滤，封包重定向和网络地址转换（NAT）等功能。

## 规则

​	在iptables防火墙管理中，是由一条条规则来组成的，规则（rules）的一般定义为**“如果数据包头符合这样的条件，就这样处理这个数据包”** 。rules存储在内核空间的信息包过滤表中，rules分别指定了源地址、目的地址、传输协议（TCP、UDP、ICMP）和服务类型（HTTP、FTP、SMTP）等，当数据包与规则匹配，iptables根据规则所定义的方法来处理这些数据包，包括accept、reject、drop等。

​	**对防火墙的配置本质上就是添加、修改或删除这些规则**。

## iptables和netfilter

​	iptables只是linux防火墙的管理工具，真正实现防火墙功能的是netfilter，它是Linux内核中实现包过滤的内部结构。

## 数据包在iptables中的流程

​	



## reference

https://www.jianshu.com/p/ee4ee15d3658 