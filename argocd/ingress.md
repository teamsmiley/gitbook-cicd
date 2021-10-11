# ingress

ERR_TOO_MANY_REDIRECTS

ingress와 서비스를 연결할때 다음 옵션을 사용한다.

```
nginx.ingress.kubernetes.io/backend-protocol: 'HTTPS'
```

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/backend-protocol: 'HTTPS'
spec:
  rules:
    - host: argocd.YourDomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 80
  tls:
    - hosts:
        - argocd.YourDomain.com
      secretName: argocd-tls
```

이제 확인해보자. 잘 될것이다.
