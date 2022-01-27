---
title: HPE 3PAR	替换故障硬盘过程
tags: 
- 存储
---

**1自动执行过程**
当系统检测到硬盘故障时，会将其标记为 degraded 状态。默然情况下，系统会自动发起 servicemag start 进程，将坏盘的数据重构到热备空间。在重构过程中，使用 showpd -s 可以看到正在 relocating，重构结束后，硬盘被标记为failed。此时换上新的硬盘，系统自动发起 servicemag resume 进程，将坏盘的数据回迁到新的硬盘
**2手动执行**
当系统没有自动执行 servicemag start 或者状态显示正常，但有存在风险的硬盘需要替换时，需要手动执行。过程如下：

1. 使用 showpd -s 命令验证磁盘状态
1. 使用 servicemag status -d 验证系统是否自动发起 servicemag start 进程及数据是否迁移完成，如果没有发起start进程，手动执行
1. 更换硬盘
1. 使用 servicemag status -d 验证系统是否自动发起 servicemag resume 进程及数据是否回迁完成，如果没有，手动执行
1. 数据回迁结束后，使用 dismisspd \<pdid\> 清除坏盘的 pdid 
