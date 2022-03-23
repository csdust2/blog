---
title: FreeBSD 存储多路径
tag:
- 运维
---

启用多路径，添加内核模块到 /boot/loader.conf 文件
```
geom_multipath_load="YES"
```
简单使用方法
手动方法
```
gmultipath create -Av name /dev/da1 /dev/da2 ...
```
自动方法
```
gmultipath label -Rv name /dev/da1 /dev/da2 ...
```
添加磁盘
```
gmultipath add -v name /dev/da3
```
移除磁盘
```
gmultipath remove -v name /dev/da3
```
参数说明
-v 输出详细信息
-A 选项 Active / Active 模式： 此模式下，所有未标记为 FAIL 的路径都可以同时处理 I / O
-R 选项 Active / Read 模式： 所有未标记为 FAIL 的路径都可以同时处理读取，但与 Acitve / Active 模式不同， 只有一个路径可以处理写入请求。
默认 Active / Passive 模式： 只有一条路径可以处理 I / O 操作，直到路径失效后切换到其他路径。
一些示例：
模式切换
```
gmultipath configure -R name
```
切换路径 （切换到下一条可用路径）
```
gmultipath rotate name
```
切换到指定路径
```
gmultipath prefer name /dev/da3
```
查看状态
```
gmultipath status name
```
销毁多路径
```
gmultipath destroy name
```

**参考文章**
[**gmultipath 手册**](https://www.freebsd.org/cgi/man.cgi?query=gmultipath&apropos=0&sektion=0&manpath=FreeBSD+13.0-RELEASE&arch=default&format=html)


