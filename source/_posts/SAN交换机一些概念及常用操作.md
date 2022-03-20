---
title: SAN 交换机一些概念及常用操作
tag:
- 存储
---

**1.端口种类**
设备侧端口类型：主要指和交换机相连的终端设备
N_Port：点对点模式的端口（用与和交换机连接）
NL_Port：仲裁环模式的端口（两个终端设备之间直连：如存储与服务器直连）

---

交换机常用端口类型：
F_Port：一种交换连接端口，用于连接两个 N_port
E_Port：扩展端口，用于和其它交换机建立互联的端口
G_Port：通用端口，并非端口的最终状态，根据连接的方式可在 F_Port 和 E_Port 之间进行切换

---

**2.ZONE**
zone 是存储交换机中一种逻辑配置。通过将指定的设备加入 zone，从而允许 zone 内设备之间互相通信。默认没有 zone 配置，设备之间都可以互相通信（不推荐如此使用，会有性能影响，也无法做定制化的管理）
zone 的实现方式有2种，一种称为硬 zone 一种为软 zone
硬 zone 基于 Domain ID 与端口号与终端配对，即使终端上的设备改变也不会影响 zone 的使用。例如更换主机的 hba 卡，交换机上无需做任何更改，但如果更换交换机端口需要更改配置
软 zone 基于终端的 wwpn / wwnn 号来配对，不关心设备接到交换机的那个端口上。当设备从一个端口移动到另一个端口时，不需要进行任何更改。但如果主机更换 hba 卡，需根据新卡的 wwn 号来更改配置。（如果有启用 NPIV 的主机，只能使用软 zone）

---

**常用操作**
alicreate：创建别名，用以给终端一个助记名，方便管理（非必须，但推荐）
例如：
```
alicreate "Nimble_p1","1,0"  //此为硬zone创建方法
alicreate "Nimble_p2","1,1"
alicreate "DBVM_p1","10:00:08:f1:ea:b"  //此为软zone创建方法
alicreate "DBVM_p2","10:00:08:f1:ea:c"
```
zonecreate：创建 zone
例如：
```
zonecreate "DBVM_p1_Nimble_p1","Nimble_p1;DBVM_p1"
zonecreate "DBVM_p2_Nimble_p2","Nimble_p2;DBVM_p2"
```
cfgcreate：创建zone的配置文件
例如：
```
cfgcreate "product","DBVM_p1_Nimble_p1;``DBVM_p2_Nimble_p2"
```
启用配置并保存
```
cfgenable "product"
cfgsave
```
