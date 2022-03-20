---
title: FreeBSD 配置网卡绑定
tag: 
- 运维
---

启用 lagg ，编辑 /boot/loader.conf 文件添加以下行
```
if_lagg_load="YES"
```
支持的聚合协议类型：
**failover**：只通过活动端口发送流量。如果主端口变为不可用，使下一个端口变为活动。第一个添加的接口为主接口， 任何之后添加的接口为失效转移设备。 默认只接受活动端口的流量，可以通过 sysctl 设置 net.link.lagg.failover_rx_all 变量为一个非零值来改变此限制
**lacp**：支持IEEE 802.1AX 链路聚合控制协议与标记协议。链路聚合组（LAG）由相同速率的端口组成，且设置为全双工。流量在端口组内平衡
**loadbalance**：根据散列协议头信息平衡活动端口上的传出流量，并接受来自任何活动端口的传入流量。此为静态设置，不会监视链路状态。散列包括 MAC 源和目标地址，以及VLAN标记（如果可用）以及IP源和目标地址
**roundrobin**：循环调度输出流量通过所有活动端口，接受任何活动端口的输入流量。使用 roundrobin 模式可能导致无序数据包到达客户端
**broadcast**：将帧发送到 LAG 的所有端口，并在 LAG 的任何端口上接收帧
**none**：什么也不做，禁用任何流量，而不会禁用lagg接口本身

配置 lagg
```
ifconfig lagg0 create
ifconfig lagg0 laggproto failover laggport em0 laggport em1 192.168.x.x netmask 255.255.255.0
```
要在重启时保留此配置，添加以下行到 /etc/rc.conf
```
ifconfig_em0="up"
ifconfig_em1="up"
cloned_interfaces="lagg0"
ifconfig_lagg0="laggproto failover laggport em0 laggport em1 192.168.x.x/24"
```

**参考文章**
[**FreeBSD 手册**](https://docs.freebsd.org/en/books/handbook/advanced-networking/#network-aggregation)
[**lagg手册**](https://www.freebsd.org/cgi/man.cgi?query=lagg)
