---
title: Linux存储多路径
tag:
- 运维
---

当存储与服务器映射时，一个存储上的卷可以有多条 I/O 路径。在操作系统上看就会有多个硬盘，但这些硬盘实际指向的是同一个卷。配置多路径软件会将这些 I/O 路径聚合，并生成由聚合路径组成的新设备，以提供冗余（主备模式）或改进的性能（轮询模式）
当被多路径控制的新设备，会在 /dev/mapper 和 /dev/ 目录中生成这些设备。所有格式为 /dev/dm-x 的设备都仅用于内部使用，管理员不应直接使用。每个多路径设备都有一个全球标识符（WWID），默认情况下会将多路径的设备名设为它的 WWID。可以通过修改配置文件的 user_friendly_names 选项，将设备别名设置为 mpath-n 的节点名称
**配置文件概述**
多路径配置文件可包含以下部分，并不是每个部分都需要
**黑名单（blacklist）**
不做为多路径的设备列表
**blacklist_exceptions**
根据 blacklist 部分的参数，列出不在黑名单中的多路径设备
**defaults**
DM 多路径的常规默认配置
**multipaths**
各独立多路径设备的特性设置。这些数值覆盖了配置文件中的 defaults 和 devices 中指定的数值
**devices**
各个存储控制器的设置。覆盖了配置文件中 defaults 部分指定的数值。如果要使用不是默认支持的存储阵列，则可能需要创建 devices 子部分。

**多路径配置设置**

| 属性 | 描述 |
| --- | --- |
| polling_interval | 以秒为单位指定两次路径检查之间的间隔。<br> 对正常工作的路径，两次检查的间隔会逐渐<br>增加到 polling_interval 的四倍。 |
| find_multipaths | 定义设置多路径的方法。如果设置为 yes，<br>DM 将不会尝试为未列入黑名单的每条路径<br>创建设备。相反，只有满足以下三个条件之一时，多路径才会创建设备：<br>1.不在黑名单的两个路径的 wwid 相同<br>2.使用 multipath 命令强制创建<br>3.一个路径的 wwid 与之前已经创建的多路径设备相同 |
| path_selector | 决定下一个 I/O 操作所使用路径的算法。可能的取值包括：<br>**round-robin 0**：循环每个路径，向每个路径发送同样数量的 I/O。此为默认值<br>**queue-lenght 0**：将下一组 I/O 发送到具有最少未处理 I/O 请求的路径<br>**service-time 0**：将下一组 I/O 发送到具有最短预计服务时间的路径，这是由未处理 I/O 的总量除以每个路径的相对流量决定的 |
| path_grouping_policy | 路径冗余策略，可能的值包括：<br>failover：在一个设备组中，只有一条路径有优先权<br>multibus：所有有效路径具有相同优先级 |
| prio | 路径优先级获取方法，可能的取值：<br>const：所有路径优先级为 1<br>emc：为 emc 阵列生成优先权<br>alua：根据 SCSI-3 ALUA 设置生成优先级，在较新的系统中推荐设置为 alua <br>ontap：为 NETAPP 生成优先级 |
| path_checker | 检测路径状态的方法，可能的取值：<br>readsector0：读取设备的第一扇区<br>tur：在设备中执行 TEST UNIT READY 命令<br>directio：直接 I/O 的方式检查链路状态。此为默认方式 |
| failback | 故障恢复的方法，可能的取值：<br>immediate：从高优先级的路径中选择一条<br>manual：手动恢复<br>followover：当故障路径恢复活跃时自动恢复  |
| rr_min_io_rq | 使用 request-based DM 指定切换到当前路径组中<br>下一个路径前路由到该路径的 I/O 请求数。<br>默认值为 1 |
| rr_weight | 路径的权重，默认值为 uniform 所有路径权重相同 |
| no_path_retry | 指定系统在禁用系统队列前，<br>尝试使用失败路径的次数，可能的取值：<br>fail：立即失败，无需排队<br>queue：在路径固定前一直排队<br>默认值为：0 |
| user_friendly_names | 如果设为 yes，即指定系统使用 /etc/multipath/bindings <br>为该多路径分配一个持久且唯一的别名，格式为 mpath-n。<br>如果设置为 no，则使用 wwid 为路径别名 |
| max_fds | multipath 及 multipathd 可打开的最大文件描述符数 |

安装及使用
```
dnf install device-mapper-multipath \\安装多路径软
modprobe dm-multipath \\安装多路径模块
modprobe dm-round-robin \\安装负载均衡模块
mpathconf --enable \\生成配置文件，默认在 /etc/multipath.conf
systemctl start multipathd \\启动多路径服务
systemctl enable multipathd \\开机自启动多路径服务
multipath -ll \\查看多路径设备
```

例子：
```
defaults {
    user_friendly_names yes
    find_multipaths no
}
blacklist {
    devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
    devnode "^hd[a-z]"
    device {
        vendor ".*"
        product ".*"
    }
}
blacklist_exceptions {
    device {
        vendor "Nimble"
        product "Server"
    }
}
devices {
    device {
        vendor "Nimble"
        product "Server"
        path_grouping_policy group_by_prio
        prio "alua"
        hardware_handler "1 alua"
        path_selector "service-time 0"
        path_checker tur
        no_path_retry 30
        failback immediate
        fast_io_fail_tmo 5
        dev_loss_tmo infinity
        rr_min_io_rq 1
        rr_weight uniform
    } 
}
```
**参考文章**
[红帽多路径配置及管理](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/dm_multipath/index)
