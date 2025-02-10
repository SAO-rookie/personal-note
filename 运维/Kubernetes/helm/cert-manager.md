# cert-manager的安装
[官方文档](https://cert-manager.io/docs/tutorials/acme/nginx-ingress/#step-6---configure-a-lets-encrypt-issuer)

1. 安装
**通过 helm 安装**
```
helm repo add jetstack https://charts.jetstack.io --force-update

helm install  cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace   --set crds.enabled=true
```
2.添加发行平台
**注意：Issuer类型只能在名称空间使用，ClusterIssuer类型没有任何限制**
创建两个Issuer或者ClusterIssuer.一个测试，一个发行
2. 先创建测试是否能连同
```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: user@example.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging
    # Enable the HTTP-01 challenge provider
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
```
3. 后创建是否可以使用
```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: user@example.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
```
3.ingress-nginx 配置自动配置ssl
**注意：根据情况使用 issuer 或 cluster-issuer**
**注意：当连通成功 就换成正式使用的**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kuard
  annotations:
    cert-manager.io/issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.example.com
    secretName: quickstart-example-tls
  rules:
  - host: example.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kuard
            port:
              number: 80
```

# 其他免费 SSL供应商
[Let's Encrypt](https://letsencrypt.org/)
[ZeroSSL](https://zerossl.com/)
[freessl](https://freessl.cn/)