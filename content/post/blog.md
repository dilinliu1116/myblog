+++
date = '2025-11-10T02:22:47Z'
draft = false
title = 'Domain-admin for k8s'
+++


# Domain-Admin 服务 Kubernetes 完整部署配置

## 前提准备
**系统环境：**
- 系统：Debian 12
- Kubernetes：v1.28
- 容器运行时：containerd v1.6.24
- 存储类：NFS（已部署 SC）

---
## 创建PVC
## 保证domain-admin服务数据的持久化，编写pvc.yaml （前提已经部署完成SC,实现了nfs共享存储
### pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: domain-admin              #   pod  name
  namespace: domain-admin     #   pod所在的NS
spec:
  accessModes:
    - ReadWriteMany           #权限为RWM
  resources:
    requests:
      storage: 1Gi
  storageClassName:   nfs-storage     #创建的sc
```
---

```bash
kubectl apply -f pvc.yaml
```

## 完整部署配置文件 `domain-admin.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: domain-admin
  namespace: domain-admin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: domain-admin
  template:
    metadata:
      labels:
        app: domain-admin
    spec:
      containers:
      - name: domain-admin
        image: mouday/domain-admin:v1.5.25
        ports:
          - containerPort: 8000
        volumeMounts:
          - name: database-db
            mountPath: /app/database
      dnsPolicy: "None"        # #dns策略，需要添加，否则不能访问内网的域名
      dnsConfig:
        nameservers:        #用于域名解析的DNS服务器列表
          - 10.96.0.10       # k8s内部的（默认的）
          - 192.168.10.1   #内网的DNS
        searches:      # 用于域名搜索的DNS域名后缀，用的是pod默认的
          - monitoring.svc.cluster.local
          - svc.cluster.local
          - cluster.local
        options:     # 配置其他可选DNS参数，用的是pod默认的
          - name: ndots
            value: "5"
      volumes:
        - name: localtime
          hostPath:
            path: /usr/share/zoneinfo/Asia/Shanghai
        - name: database-db           #上面创建pvc的名字
          persistentVolumeClaim:
            claimName: domain-admin

---
apiVersion: v1
kind: Service
metadata:
  name: domain-admin
  namespace: domain-admin
spec:
  selector:
    app: domain-admin
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 8000
```
## 部署ingress，实现域名访问
```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: domain-admin-ingress
  namespace: domain-admin
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "8m"
spec:
  tls:
  - hosts:
      - "www.domain-admin.io"
    secretName: "your-tls-secret"  # 替换为实际的 TLS Secret 名称
  rules:
  - host: "www.domain-admin.io"
    http:
      paths:
      - path: "/"
        pathType: Prefix
        backend:
          service:
            name: domain-admin
            port:
              number: 8000
```

## 配置说明

### 需要替换的变量
- **镜像版本**：`mouday/domain-admin:v1.5.25`（根据实际版本调整）
- **TLS Secret**：`your-tls-secret`（替换为实际的证书 Secret 名称）
- **域名**：`www.domain-admin.io`（替换为实际域名）
- **内网 DNS**：`192.168.10.1`（根据实际网络环境调整）

### 可选配置项
- **资源限制**：可添加 `resources` 配置 CPU 和内存限制
- **健康检查**：可添加 `livenessProbe` 和 `readinessProbe`
- **环境变量**：可根据需要添加 `env` 配置

[专业数学建模指导点击这里](https://item.taobao.com/item.htm?ft=t&id=959173935001)
