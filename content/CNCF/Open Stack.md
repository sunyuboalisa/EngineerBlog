---
title: Open Stack
draft: false
tags:
---
# DevStack 部署 
## 环境
- 国内需要配置代理
```sh
# 配置会话临时代理
export http_proxy=http://127.0.0.1:1080
export https_proxy=http://127.0.0.1:1080
export no_proxy=localhost,127.0.0.1,::1

# 配置 git 代理
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy http://127.0.0.1:1080

# 取消 git 代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```
- os 要求：Ubuntu 24.04  8g 4c
## 安装
主机最好是纯净环境，因为 devstack 需要不少软件和端口，有可能和现有环境冲突，造成安装失败。
```sh
# 配饰 stack 用户
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo chmod +x /opt/stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo -u stack -i
# 拉取代码
git clone https://opendev.org/openstack/devstack
cd devstack
```
接下来需要创建 local.conf 配置文件
```
[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```
最后开始安装
```
./stack.sh
```

# 实验 
## 创建虚拟机并测试连接
### 环境
- Ubuntu 24.04.02
- DevStack

### 实验步骤
```shell
# 如果没有 ssh key，需要提前创建
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
# 加载 openstack 环境
. openrc
# 创建 keypair
openstack keypair create --public-key ~/.ssh/id_rsa.pub test-keypair
# 查看 flavor 和 image
openstack flavor list
openstack image list
# 创建 vm
openstack server create   --flavor m1.small   --image cirros-0.6.3-x86_64-disk   --network private   --key-name test-keypair   test-server
# 开启 SSH 和 ICMP
fip_id=$(openstack floating ip create public -f value -c id)
openstack server add floating ip test-server ${fip_id}
openstack security group rule create --proto icmp --dst-port 0 default
openstack security group rule create --proto tcp --dst-port 22 default
# 查看虚拟机
openstack server list
# 连接刚刚创建的虚拟机
openstack server ssh test-server -- -l cirros
```
![[Pasted image 20250808091844.png]]
### 总结
- 优点：通过 openstack 可以很好的利用计算资源，统一管理虚拟机、网络、存储等。
- 缺点：有点过于复杂，上手难度较高，对各种基础知识都要有一定深度的理解。