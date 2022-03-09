---
title: VCSA 取消无效的链接模式
tag:
- 运维
---

当有多余的 VCSA 想要取消链接模式的时候可以使用以下方法：（适用于7.0版本）

1. 使用 SSH 连接到 Platform Services Controller  
2. 创建包含已在 Platform Services Controller 中注册的服务的列表的文本文件 
```
/usr/lib/vmware-lookupsvc/tools/lstool.py list --url http://localhost:7090/lookupservice/sdk --type vcenterserver > /tmp/psc_services.txt --no-check-cert
```

3. 打开生成的文件，查找需要删除的 VCSA 的 URL 地址，对应的 Service ID
   文件中，会看到类似以下内容的输出
```
Name: AboutInfo.vpx.name
Description: AboutInfo.vpx.name
Service Product: com.vmware.cis
Service Type: vcenterserver
Service ID: 1dbc3e9f-626d-4314-8731-ca744a0d9f4b
Site ID: home
Node ID: d3eba55a-d4df-11e4-b3f7-000c2987c143
Owner ID: vpxd-2752b8d1-e68b-49f8-8c92-ce3f042bf487@vsphere.local
Version: 6.0
Endpoints:
Type: com.vmware.cis.workflow
Protocol: vmomi
URL: http://vcsa2.domain.local:8088
```

4. 删除失效的条目
```
/usr/lib/vmware-lookupsvc/tools/lstool.py unregister --url http://localhost:7090/lookupservice/sdk --id 6ae3bf1a-9318-4a33-b2cb-d2eaa7a306c5 --user 'administrator@vsphere.local' --password 'VMware123!' --no-check-cert
```
其中 ID ，用户名，密码根据自己的情况更改

参考文章
[VMware知识库](https://kb.vmware.com/s/article/2050273?lang=zh_CN)
