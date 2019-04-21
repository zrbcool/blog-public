### 目录结构
```
.
├── customized\ ingress\ dashboard.json #监控grafana dashboard
├── tap-revision-ing.ingress.yaml #ingress定义demo
├── xwz-ingress-controller-leader.configmap.yaml #configmap
├── xwz-nginx-configuration.configmap.yaml #configmap nginx配置
├── xwz-nginx-ingress-controller.deployment.yaml #ingress controller的deployment文件
├── xwz-nginx-ingress-lb.service.yaml #service定义
├── xwz-nginx-template #自定义的template configmap的配置文件内容
│   └── nginx.tmpl
├── xwz-tcp-services.configmap.yaml #configmap nginx配置
└── xwz-udp-services.configmap.yaml #configmap nginx配置

```

创建configmap
```
### create xwz-nginx-template configmap
kubectl create configmap xwz-nginx-template --from-file=xwz-nginx-template -n kube-system
```
