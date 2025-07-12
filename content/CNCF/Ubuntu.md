---
title: Ubuntu
draft: false
tags:
---
 如果你想改成静态 IP，基于你的配置模板，可以调整成这样：

```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no                # 关闭 DHCP
      addresses:
        - 192.168.1.100/24    # 你的静态IP和掩码
      gateway4: 192.168.1.1    # 网关地址
      nameservers:
        addresses:
          - 8.8.8.8
          - 114.114.114.114


```



**操作步骤：**

1. 编辑配置文件：

`sudo nano /etc/netplan/01-netcfg.yaml`

2. 替换为上面内容（修改 IP 和网关为你网络对应的地址）。
3. 应用配置：

`sudo netplan apply`

4. 检查 IP 是否生效：
`ip a show ens33 ping 8.8.8.8`