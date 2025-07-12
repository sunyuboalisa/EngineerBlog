## 为 Docker 引擎配置代理（针对 pull、push 镜像等）

适用于 Docker CLI、`docker pull`、`docker build` 中拉取镜像等操作。

### Linux 系统（如 Ubuntu、CentOS）配置步骤：

1. 创建配置目录（如未存在）：
    `sudo mkdir -p /etc/systemd/system/docker.service.d`
2. 创建代理配置文件：
    `sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf`
3. 写入如下内容（将代理地址替换为你的）：
    `[Service] Environment="HTTP_PROXY=http://127.0.0.1:1080" Environment="HTTPS_PROXY=http://127.0.0.1:1080" Environment="NO_PROXY=localhost,127.0.0.1,192.168.0.0/16"`
    
4. 重新加载并重启 Docker 服务：
    `sudo systemctl daemon-reexec sudo systemctl daemon-reload sudo systemctl restart docker`
    
5. 验证代理是否生效：
    `systemctl show --property=Environment docker`

### 1. 在终端中查看环境变量

在 macOS / Linux / WSL / Git Bash / PowerShell 等终端中输入：

bash

复制编辑

`env | grep -i proxy`

systemctl show --property=Environment docker---
title: Docker
draft: false
tags:
---
 