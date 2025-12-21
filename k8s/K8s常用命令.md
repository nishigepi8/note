## Pod

```sh
#获取指定namespace下的Pod
kubectl get pod -n ${namespace}

#获取所有Pod
kubectl get pods

#获取Pod的详细信息
kubectl get pod ${pod-name} -o wide
kubectl get pod ${pod-name} -o yaml
```

## Service

```sh
#获取指定namespace下的Service
kubectl get svc -n ${namespace}

#获取所有Service
kubectl get services

#获取Service的详细信息
kubectl describe svc ${service-name}
kubectl get svc ${service-name} -o wide
kubectl get svc ${service-name} -o yaml
```



## Deployment

```sh
#获取指定namespace下的Deployment
kubectl get deploy -n ${namespace}

#获取所有Deployment
kubectl get deployments

#获取Deployment的详细信息
kubectl get deploy ${deployment-name} -o wide
kubectl get deploy ${deployment-name} -o yaml

#删除部署
kubectl delete deployment/deployment_name -n namespace
```



## Ingress-Nginx

### 配置文件

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress 
metadata:
  name: ingress-http
  namespace: default
  annotations: 
    kubernetes.io/ingress.class: "nginx"
spec:
  rules: # 规则
  - host: auth.olang.com # 指定的监听的主机域名，相当于 nginx.conf 的 server { xxx }
    http: # 指定路由规则
      paths:
      - path: /
        pathType: Prefix # 匹配规则，Prefix 前缀匹配 it.nginx.com/* 都可以匹配到
        backend: # 指定路由的后台服务的 service 名称
          service:
            name: auth-service # 服务名
            port:
              number: 3000 # 服务的端口
```

