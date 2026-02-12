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
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: kong
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-svc
  namespace: kong
spec:
  ports:
    - port: 5432
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: kong
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16
          env:
            - name: POSTGRES_USER
              value: "kong"
            - name: POSTGRES_PASSWORD
              value: "kong_pass" 
            - name: POSTGRES_DB
              value: "kong"
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: pg-data
              mountPath: /var/lib/postgresql/data
              subPath: postgres 
      volumes:
        - name: pg-data
          persistentVolumeClaim:
            claimName: postgres-pvc
 # 将上面的放到一个postgres.yaml文件中
kubectl create -f postgres.yaml
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.3.0/standard-install.yaml

tar -zxvf helm-v4.0.5-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
helm repo add kong https://charts.konghq.com
helm repo update

# 安装 Kong Ingress
export HOST_IP="192.168.0.10"
helm upgrade --install kong kong/ingress -n kong \
  --set gateway.admin.enabled=true \
  --set gateway.admin.http.enabled=true \
  --set gateway.admin.type=NodePort \
  --set gateway.admin.http.nodePort=30001 \
  --set gateway.manager.enabled=true \
  --set gateway.manager.type=NodePort \
  --set gateway.manager.http.nodePort=30256 \
  --set gateway.env.admin_listen='0.0.0.0:8001\, 0.0.0.0:8444 ssl' \
  --set gateway.env.admin_gui_url="http://$HOST_IP:30256" \
  --set gateway.env.admin_gui_api_url="http://$HOST_IP:30001" \
  --set gateway.env.admin_gui_cors_origins="*" \
  --set gateway.env.database=postgres \
  --set gateway.env.pg_host=postgres-svc \
  --set gateway.env.pg_user=kong \
  --set gateway.env.pg_password=kong_pass \
  --set gateway.env.pg_database=kong \
  --set postgresql.enabled=false \
  --set controller.ingressController.env.feature_gates="GatewayAlpha=true"
  
helm uninstall kong -n kong
```


## Linkerd

```sh

export LINKERD2_VERSION=edge-25.10.7
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh
export PATH=$HOME/.linkerd2/bin:$PATH
linkerd version

linkerd check --pre

linkerd install --crds | kubectl apply -f -
linkerd install | kubectl apply -f -
linkerd check

linkerd viz install | kubectl apply -f -

```

## Flannel

