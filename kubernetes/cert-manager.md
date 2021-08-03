# Cert-Manager

ssl을 생성해주는 툴

ClusterIssuer : 클러스터 전체에서 한개의 issuer를 사용가능하게

Issuer : namespace마다 하나의 issuer를 생성해서 사용하게

예전에는 certificate를 따로 만들고 이 이름을 ingress에 넣어주어서 생성을 햇으나 anotation이 생겼다.

{% embed url="https://cert-manager.io/docs/usage/ingress/" %}

- `cert-manager.io/issuer`: the name of an `Issuer` to acquire the certificate required for this `Ingress`. The Issuer _must_ be in the same namespace as the `Ingress` resource.
- `cert-manager.io/cluster-issuer`: the name of a `ClusterIssuer` to acquire the certificate required for this `Ingress`. It does not matter which namespace your `Ingress` resides, as `ClusterIssuers` are non-namespaced resources.

이걸 사용하면 하나로 처리가 가능하다.

```text
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: "dns-issuer-aws-live"
spec:
  tls:
    - hosts:
        - test.xgridcolo.com
      secretName: test-tls
  rules:
    - host: test.xgridcolo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: ngnix-service
                port:
                  number: 80

```
