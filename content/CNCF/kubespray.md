---
title: kubespray
draft: false
tags:
---
 ## 环境准备

### 准备五台节点

control  3台

2c 4g 50g 桥接

work 2台

2c 8g 100g 桥接

  

OS：ubuntu-22.04.5-live-server-amd64.iso (Ubuntu Jammy 22.04(LTS))

  

用户：user

密码：#Alisa96312

  

启动后每个节点进行一下配置

#### 静态IP

内容是

```

sudo nano /etc/netplan/01-netcfg.yaml

# 内容是

network:

    version: 2

    renderer: networkd

    ethernets:

        enp0s3:

            addresses:

              - 192.168.1.118/18

  

sudo netplan apply

```

  
  

#### APT 内部仓库

```

sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

  

sudo nano /etc/apt/sources.list

  

# 内容填写

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to          # newer versions of the distribution.                                              deb http://domain/repository/ubuntu-2204 jammy main restricted                                                    

# deb-src http://domain/repository/ubuntu-2204 jammy main restricted      ## Major bug fix updates produced after the final release of the                   ## distribution.                                                                   deb http://domain/repository/ubuntu-2204 jammy-updates main restricted    

  

# deb-src http://domain/repository/ubuntu-2204 jammy-updates main restricted                                      

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu        ## team. Also, please note that software in universe WILL NOT receive any          ## review or updates from the Ubuntu security team.                                deb http://domain/repository/ubuntu-2204 jammy universe                  

  

# deb-src http://domain/repository/ubuntu-2204 jammy universe             deb http://domain/repository/ubuntu-2204 jammy-updates universe          

# deb-src http://domain/repository/ubuntu-2204 jammy-updates universe     ## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu        ## team, and may not be under a free licence. Please satisfy yourself as to        ## your rights to use the software. Also, please note that software in             ## multiverse WILL NOT receive any review or updates from the Ubuntu               ## security team.                                                                  deb http://domain/repository/ubuntu-2204 jammy multiverse                 # deb-src http://domain/repository/ubuntu-2204 jammy multiverse           deb http://domain/repository/ubuntu-2204 jammy-updates multiverse         # deb-src http://domain/repository/ubuntu-2204 jammy-updates multiverse   ## N.B. software from this repository may not have been tested as                  ## extensively as that contained in the main release, although it includes         ## newer versions of some applications which may provide useful features.          ## Also, please note that software in backports WILL NOT receive any review        ## or updates from the Ubuntu security team.                                       deb http://domain/repository/ubuntu-2204 jammy-backports main restricted universe multiverse                      

# deb-src http://domain/repository/ubuntu-2204 jammy-backports main restricted universe multiverse                

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted              # deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted        deb http://security.ubuntu.com/ubuntu/ jammy-security universe                     # deb-src http://security.ubuntu.com/ubuntu/ jammy-security universe               deb http://security.ubuntu.com/ubuntu/ jammy-security multiverse                   # deb-src http://security.ubuntu.com/ubuntu/ jammy-security multiverse

  

sudo apt update

  

```

#### SSH 环境

```

sudo apt update

# 如果提示已存在客户端，执行下面的

sudo apt remove openssh-client

sudo apt install openssh-server

```

  

#### 加载内核模块

```

# 连接跟踪相关

sudo modprobe nf_conntrack

sudo modprobe nf_conntrack_netlink

sudo modprobe nf_defrag_ipv4

sudo modprobe nf_defrag_ipv6

  

# iptables 基础模块

sudo modprobe ip_tables

sudo modprobe x_tables

  

# ipset 支持

sudo modprobe ip_set

sudo modprobe xt_set

  

# NAT/iptables 扩展

sudo modprobe xt_nat

sudo modprobe xt_MASQUERADE

sudo modprobe xt_conntrack

sudo modprobe xt_tcpudp

sudo modprobe xt_LOG

sudo modprobe xt_addrtype

sudo modprobe ipt_REJECT

sudo modprobe xt_limit

sudo modprobe xt_hl

  

# Netfilter 核心通信模块

sudo modprobe nfnetlink

```

  
  

### 准备一台部署机

4c 16g 200g 桥接

  

OS：pop-os_22.04_amd64_nvidia_54.iso

  

用户：user

密码：#Alisa96312

  

启动后进行以下配置

静态IP

UI 中去有线设置上面填写静态IP即可

APT仓库

```

sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

  

sudo nano /etc/apt/sources.list

  

# 内容填写

  

deb http://domain/repository/ubuntu-2204/ jammy main restricted

sudo apt update

```

#### Ansible

python

pyenv

vs code

vs code extension

  

```

docker load -i

alias docker-ansible-cli='docker run --rm -it =v $(pwd):/ansible -v ~/.ssh/id_rsa:/root/.ssh/id_rsa --workdir=/ansible willhallonline/ansible:2.17-ubuntu-22.04 /bin/sh'

```

#### Docker

离线安装

```

# 删除有冲突的包

for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

  

# 将准备好的离线文件放在一个文件夹中，切换当其目录下进行安装

sudo dpkg -i ./*.deb

  

# 验证安装

docker version

docker compose version

  

# 创建docker组 & 添加用户

sudo groupadd docker

sudo usermod -aG docker $USER

  

# 注销重新登陆 | 激活对组的更改

newgrp docker

  

# 配置自启动

sudo systemctl enable docker.service

sudo systemctl enable containerd.service

```

  
  
  

## 部署

  

### 脚本依赖环境：

拉取 kubespray v2.8 源文件，并解压到 kubespray v2.8 目录下

切换到 kubespary v2.8 目录，创建 python venv 环境

  

```

python -m venv localenv

source localenv/bin/activate

  

pip install -i http://domain/repository/pypi-thu/simple -r requirements.txt --trusted-host domain

  

之后执行

./generate_list.sh

  

# 然后修改 manage-offline-container-images.sh ，里面添加 IMAGES_FROM_FILE 变量指向images.list

IMAGES_FROM_FILE="{TEMP_DIR}/images.list"

  

# 下载离线镜像

./manage-offline-container-images.sh create

  

# 下载离线文件

./manage-offline-files-plus.sh create

  

# 启动镜像服务器，并注册所有下载好的镜像

./manage-offline-container-images.sh register

  

#启动离线文件服务器

./manage-offline-files-plus.sh register

  

```

  

### 离线环境

静态文件服务器 & 镜像服务器

首先得确保ansible环境好了

下载所有静态文件和镜像文件

```

# 执行后会在temp目录中生成 files.list 和 images.list 文件。里面有所有需要的离线文件

# 可以用脚本在一个能联网的主机上下载这些文件，主机上要配置代理的，而且docker也要配置代理，有访问国外资源

./generate_list.sh

  

# 生成好之后

# 修改manage-offline-container-images.sh脚本中的变量

DESTINATION_REGISTRY=你自己的registry的endpoint

IMAGES_FROM_FILE=你自己的镜像列表文件

例如DESTINATION_REGISTRY=domain:5000

IMAGES_FROM_FILE="${TEMP_DIR}/images.list"

  
  

# 运行脚本，生成离线镜像

./manage-offline-container-images.sh create

  

# 运行脚本，生成离线文件

./manage-offline-files-plus.sh create

  

# 运行脚本，运行registry 并推送离线镜像

./manage-offline-container-images.sh register

  

# 运行脚本，加载 nginx 并开启离线文件服务器

./manage-offline-files-plus.sh register

  

```

修改 offline文件夹下的 docker-daemon.json 文件

  

```

{ "insecure-registries":["domain:5000"] }

```

### SSH 环境

  

```

ssh-keygen -t rsa -b 4096

ssh-copy-id user@remotehost

  

# 如果遇到权限问题，在远程机器上修改 authorized_keys 文件

chmod 600 .ssh/authorized_keys

```

  

### Inventory

修改自己的 inventory 文件

  

```

# This inventory describe a HA typology with stacked etcd (== same nodes as control plane)

# and 3 worker nodes

# See https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

# for tips on building your # inventory

# Configure 'ip' variable to bind kubernetes services on a different ip than the default iface

# We should set etcd_member_name for etcd cluster. The node that are not etcd members do not need to set the value,

# or can set the empty string value.

  

[kube_control_plane]

node1 ansible_host=192.168.1 ip=192.168.1 etcd_member_name=etcd1

node2 ansible_host=192.168.1 ip=192.168.1 etcd_member_name=etcd2

node3 ansible_host=192.168.1 ip=192.168.1 etcd_member_name=etcd3

  

[etcd:children]

kube_control_plane

  

[kube_node]

node4 ansible_host=192.168.1 ip=192.168.1

node5 ansible_host=192.168.1 ip=192.168.1

  

[all:vars]

ansible_user=user

ansible_become=true

  

```

修改 offline.yaml 文件

```

---

  

## Global Offline settings

  

### Private Container Image Registry

registry_addr: "domain:5000"

registry_host: "http://domain:5000"

files_repo: "http://domain:8080"

  

# Insecure registries for containerd

containerd_registries_mirrors:

    - prefix: "{{ registry_addr }}"

    mirrors:

        - host: "{{ registry_host }}"

          capabilities: ["pull", "resolve"]

          skip_verify: true

  

## Container Registry overrides

kube_image_repo: "{{ registry_addr }}"

gcr_image_repo: "{{ registry_addr }}"

github_image_repo: "{{ registry_addr }}"

docker_image_repo: "{{ registry_addr }}"

quay_image_repo: "{{ registry_addr }}"

  

## Kubernetes components

kubeadm_download_url: "{{ files_repo }}/dl.k8s.io/release/v{{ kube_version }}/bin/linux/{{ image_arch }}/kubeadm"

kubectl_download_url: "{{ files_repo }}/dl.k8s.io/release/v{{ kube_version }}/bin/linux/{{ image_arch }}/kubectl"

kubelet_download_url: "{{ files_repo }}/dl.k8s.io/release/v{{ kube_version }}/bin/linux/{{ image_arch }}/kubelet"

## Two options - Override entire repository or override only a single binary.

## [Optional] 2 - Override a specific binary

## CNI Plugins

cni_download_url: "{{ files_repo }}/github.com/containernetworking/plugins/releases/download/v{{ cni_version }}/cni-plugins-linux-{{ image_arch }}-v{{ cni_version }}.tgz"

  

## cri-tools

crictl_download_url: "{{ files_repo }}/github.com/kubernetes-sigs/cri-tools/releases/download/v{{ crictl_version }}/crictl-v{{ crictl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"

  

## [Optional] etcd: only if you use etcd_deployment=host

etcd_download_url: "{{ files_repo }}/github.com/etcd-io/etcd/releases/download/v{{ etcd_version }}/etcd-v{{ etcd_version }}-linux-{{ image_arch }}.tar.gz"

get_helm_url: "{{ files_repo }}/get.helm.sh"

  

# [Optional] Calico: If using Calico network plugin

calicoctl_download_url: "{{ files_repo }}/github.com/projectcalico/calico/releases/download/v{{ calico_ctl_version }}/calicoctl-linux-{{ image_arch }}"

  

# [Optional] Calico with kdd: If using Calico network plugin with kdd datastore

calico_crds_download_url: "{{ files_repo }}/github.com/projectcalico/calico/archive/v{{ calico_version }}.tar.gz"

  

# [Optional] Cilium: If using Cilium network plugin

# ciliumcli_download_url: "{{ files_repo }}/github.com/cilium/cilium-cli/releases/download/v{{ cilium_cli_version }}/cilium-linux-{{ image_arch }}.tar.gz"

  

# [Optional] helm: only if you set helm_enabled: true

helm_download_url: "{{ files_repo }}/get.helm.sh/helm-v{{ helm_version }}-linux-{{ image_arch }}.tar.gz"

  

# [Optional] crun: only if you set crun_enabled: true

# crun_download_url: "{{ files_repo }}/github.com/containers/crun/releases/download/{{ crun_version }}/crun-{{ crun_version }}-linux-{{ image_arch }}"

  

# [Optional] kata: only if you set kata_containers_enabled: true

# kata_containers_download_url: "{{ files_repo }}/github.com/kata-containers/kata-containers/releases/download/{{ kata_containers_version }}/kata-static-{{ kata_containers_version }}-{{ image_arch }}.tar.xz"

  

# [Optional] cri-dockerd: only if you set container_manager: docker

# cri_dockerd_download_url: "{{ files_repo }}/github.com/Mirantis/cri-dockerd/releases/download/v{{ cri_dockerd_version }}/cri-dockerd-{{ cri_dockerd_version }}.{{ image_arch }}.tgz"

  

# [Optional] runc: if you set container_manager to containerd or crio

runc_download_url: "{{ files_repo }}/github.com/opencontainers/runc/releases/download/v{{ runc_version }}/runc.{{ image_arch }}"

  

# [Optional] cri-o: only if you set container_manager: crio

# crio_download_base: "download.opensuse.org/repositories/devel:kubic:libcontainers:stable"

# crio_download_crio: "http://{{ crio_download_base }}:/cri-o:/"

# crio_download_url: "{{ files_repo }}/storage.googleapis.com/cri-o/artifacts/cri-o.{{ image_arch }}.v{{ crio_version }}.tar.gz"

# skopeo_download_url: "{{ files_repo }}/github.com/lework/skopeo-binary/releases/download/v{{ skopeo_version }}/skopeo-linux-{{ image_arch }}"

  

# [Optional] containerd: only if you set container_runtime: containerd

containerd_download_url: "{{ files_repo }}/github.com/containerd/containerd/releases/download/v{{ containerd_version }}/containerd-{{ containerd_version }}-linux-{{ image_arch }}.tar.gz"

nerdctl_download_url: "{{ files_repo }}/github.com/containerd/nerdctl/releases/download/v{{ nerdctl_version }}/nerdctl-{{ nerdctl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"

  

```

### 开始部署

  

```

ansible-playbook -i inventory/mycluster/inventory.ini cluster.yaml -K

  

# 如果不是root用户可能需要copy下admin.conf文件

mkdir ~/.kube

sudo cp /etc/kubernetes/admin.conf ~/.

kube/config

sudo chown $(id -u):$(id -g) ~/.kube/config

```

## 管理

  
  
  

## 附录

  

表1.1 IP列表

  

| IP            | 主机名   | 备注           |

| ------------- | ----- | ------------ |

| domain |       | 内网机          |

| domain |       | 虚拟机-部署机      |

| 192.168.1 | node1 | 虚拟机-k8s控制节点1 |

| 192.168.1 | node2 | 虚拟机-k8s控制节点2 |

| 192.168.1 | node3 | 虚拟机-k8s控制节点3 |

| 192.168.1 | node4 | 虚拟机-k8s工作节点1 |

| 192.168.1 | node5 | 虚拟机-k8s工作节点2 |

  

表1.2 域名列表

  

| IP            | 域名  | 备注  |

| ------------- | --- | --- |

| domain |     |     |

| domain |     |     |

| 192.168.1|     |     |

| 192.168.1 |     |     |

| 192.168.1 |     |     |

| 192.168.1 |     |     |

| 192.168.1 |     |     |

  
  

参考

  

## FAQ

部署后遇到kubectl get nodes 提示

在控制节点上执行

```

# 如果不是root用户可能需要copy下admin.conf文件

mkdir ~/.kube

sudo cp /etc/kubernetes/admin.conf ~/.kube/config

sudo chown $(id -u):$(id -g) ~/.kube/config

```

  

部署后遇到 calico 组件没有成功启动

  

```

如果是 hygon amd cpu 那么恭喜你中奖了，不兼容

  
  

```