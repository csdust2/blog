---
title: Linux 磁盘扩容
tag:
- 运维
---

基于 LVM 的磁盘扩容一般分为 2 种情况：

1. 通过添加新的硬盘到卷组
1. 扩容现有的磁盘空间。（比如原逻辑磁盘为 3T，现在扩容为 5T）

对于第一种情况，对每一个，新添加的硬盘创建 PV，并将新建的 PV 加入到 VG，然后对 LV 做扩容，最后对文件系统做扩容。大致过程如下：
```
//将硬盘转为 PV 格式，以便 LVM 使用（推荐使用全盘的分区，而不是整个硬盘）
pvcreate /dev/sd{b1,c1,d1}
//将新硬盘添加到卷组（vg_db）
vgextend vg_db /dev/sd{b1,c1,d1}
//扩容指定的 LV
lvextend -L + 200G  /dev/vg_db/data
//扩容文件系统（只针对 xfs 文件系统）
xfs_growfs /dev/vg_db/data
```
对于第二种情况，将剩余空间加入到最后一个分区，扩容 PV，扩容 LV，最后文件系统扩容。过程如下：
```
//安装 growpart 用于分区扩容
yum install cloud-utils-growpart
//将剩余空间加入到最后一个分区（假设为 sda3)。*** growpart 只能扩容最后一分区
growpart /dev/sda 3
//扩容 PV
pvresize /dev/sda3
//扩容 LV *** 使用此方法扩容，无法使用 100%FREE 来使用全部空间，原因没找到
lvextend -L + 200G  /dev/vg_db/data
//扩容文件系统
xfs_growfs /dev/vg_db/data
```
