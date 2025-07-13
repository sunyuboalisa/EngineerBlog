---
title: k8s
draft: false
tags:
---
 # 核心概念
## Conatiner

## Workloads
### Pods
### Workload Management
Deployment
ReplicaSet
StatefulSet
DaemonSet
Job
Automatic



## Services & Load Balancing & Networking
Service
Ingress
Ingress Controller
Gateway API

## Storage

### Volumes
configMap
hostPath
local

## Configuration

## TLS

## 编排


# Plugin

## Kong Ingress Controller
```yaml
# 安装Gateway API
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml

# 创建Gateway 和 GatewayClass
echo "
---
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
 name: kong
 annotations:
   konghq.com/gatewayclass-unmanaged: 'true'

spec:
 controllerName: konghq.com/kic-gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
 name: kong
spec:
 gatewayClassName: kong
 listeners:
 - name: proxy
   port: 80
   protocol: HTTP
   allowedRoutes:
     namespaces:
        from: All
" | kubectl apply -f -

# 安装 Kong Ingress

helm install kong kong/ingress -n kong --create-namespace 
```


## Linkerd

```sh

#导出yaml安装文件，用于配置离线镜像
linkerd viz install > linkerd-viz.yaml
#修改 linkerd-viz.yaml ，确保能离线下载镜像
#安装
kubectl apply -f linkerd-viz.yaml
#检查
linkerd viz check

#运行viz
linkerd viz dashboard
#或者后台运行
linkerd viz dashboard &
#成功后会输出一个地址

# 暴露端口，用于测试
kubectl port-forward -n linkerd-viz svc/web 8084:8084 --address 0.0.0.0

# 清理测试痕迹
ps aux | grep "linkerd viz dashboard"
ps aux | grep "kubectl port-forward"
kill xxxx

```