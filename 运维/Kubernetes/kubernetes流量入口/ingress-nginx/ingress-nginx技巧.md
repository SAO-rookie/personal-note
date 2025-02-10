[官方文档](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
# ingress-nginx一个域名下配置多个后端服务
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: server-dns
spec:
  rules:
  - host: www.dasdasd.cn
    http:
      paths:
      - path: /gtz_project
        pathType: Prefix
        backend:
          service:
            name: gtz-project-server
            port:
              number: 80
      - path: /human
        pathType: Prefix
        backend:
          service:
            name: gtz-human-server
            port:
              number: 80
      - path: /vehicle
        pathType: Prefix
        backend:
          service:
            name: vehicle-server
            port:
              number: 80
  ingressClassName: nginx
```
使用上面的yaml文件配置ingress-nginx部署成功后，使用路径加请求参数访问不同的服务会出现404情况原因请参考[官方文档](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#rewrite)
所以我们使用以下解决方案
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: server-dns
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2  # 加上重写注解
spec:
  rules:
  - host: test-survey.changtech.cn
    http:
      paths:
      - path: /gtz_project(/|$)(.*) # 加上匹配规则
        pathType: Prefix
        backend:
          service:
            name: gtz-project-server
            port:
              number: 80
      - path: /human(/|$)(.*) # 加上匹配规则
        pathType: Prefix
        backend:
          service:
            name: gtz-human-server
            port:
              number: 80
      - path: /vehicle(/|$)(.*) # 加上匹配规则
        pathType: Prefix
        backend:
          service:
            name: vehicle-server
            port:
              number: 80
  ingressClassName: nginx
```
# ingress-nginx手机端跳pc端配置
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/server-snippet: >-
      if ($http_user_agent ~* "(Android|webOS|iPhone|iPod|BlackBerry)")
      {         rewrite ^($|/) https://l-mobile.cn$1
      permanent;       }
  name: lb-pc
  namespace: default
spec:
  ingressClassName: nginx
  rules:
    - host: lb.*.cn
      http:
        paths:
          - backend:
              service:
                name: lz-vue-service
                port:
                  number: 80
            path: /
            pathType: Prefix


```