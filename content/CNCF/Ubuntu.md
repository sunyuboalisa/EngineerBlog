---
title: Ubuntu
draft: false
tags:
---
配置不需要sudo密码
```
sudo visudo
alisa ALL=(ALL) NOPASSWD: ALL
```

 如果你想改成静态 IP，基于你的配置模板，可以调整成这样：

```yaml
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
# SSH
添加ssh
```sh 
#安装ssh，（ubuntu 默认安装了openssh-client）
sudo apt install openssh-server
#配置ssh
sudo sshd -t -f /etc/ssh/sshd_config
sudo systemctl restart ssh.service

#其他客户端配置ssh
#生成公钥密钥
ssh-keygen -t rsa
#将公钥复制到ssh服务器中
ssh-copy-id {username}@{remotehost}
#在文件权限不对的时候可以执行下面，
chmod 600 .ssh/authorized_keys

#其他应用导入ssh，如git
ssh-import-id <username-on-remote-service>
```
# Network
ubuntu 下主要有两种网络管理工具
- **NetworkManager**
- **systemd-networkd**
网络配置渲染器
- [Netplan](https://netplan.io/)
DNS解析器
- **systemd-resolved**

查看端口是否被占用
```sh
sudo ss -tulnp | grep :80
#或者
sudo lsof -i :80
```

编码
```
执行 cat -A myconfig.conf 的输出显示每行结尾是 $，这说明你的文件行尾是 **Windows 风格回车换行（CRLF）**，而不是 Linux 期望的 **LF**。
sudo apt install dos2unix  # 如果没安装

dos2unix myconfig.conf

#或者
vim myconfig.conf
# 打开后输入命令
:set ff=unix
:wq
```

配置文件用二维码表示
```
apt install qrencode
qrencode -o ios.png < ios.conf
```

vpn
```sh
  sudo apt install wireguard
  umask 077
  wg genkey > privatekey
  wg pubkey < privatekey > publickey
  wg genkey | tee client_private.key | wg pubkey > client_public.key
  sudo ip link add dev wg0 type wireguard
  sudo ip address add dev wg0 10.6.0.1/24
  sudo wg set wg0 listen-port 51820 private-key ./privatekey
  sudo wg set wg0 peer $(cat client_public.key) allowed-ips 10.6.0.2/32
  sudo ip link set up dev wg0
  nano ios.conf
  # 内容参考下方例子
  # 可选优化，将配置文件用二维码替代
  sudo apt install qrencode
  qrencode -o ios.png < ios.conf
  #之后手机端可以安装wireguard扫描添加vpn服务器
```
ios.conf内容
```
[Interface]
PrivateKey = iB4CsZg5XOW9//fxTFGz4hc37FIe1+S3DkPPlS0PaWo=  #客户端私钥
Address = 10.6.0.2/32 #客户端地址
DNS = 1.1.1.1

[Peer]
PublicKey = hbc9Wny8ouwCjtwpGQuMYcVlwAXsvnqfbmzq4vg5azw= #服务器公钥
Endpoint = 192.168.1.70:51820 #实际服务器ip，通常是公网ip
AllowedIPs = 10.6.0.0/24 #服务器分配的子网 CIDR
PersistentKeepalive = 25
```