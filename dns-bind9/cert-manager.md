# Cert-Manager

## create ClusterIssuer

생성해둔 키를 base64로 인코딩한다.

echo -n 'KzqRA3OnnSxx3Awp9m8Pt' | base64

이 값을 사용하여 secret을 만든후 cluster issuer를 생성한다.

tsigKeyName과 tsigAlgorithm , nameserver를 정확히 적어준다.

```yml

apiVersion: v1
kind: Secret
metadata:
  name: tsig-secret
type: Opaque
data:
  tsig-secret-key: S3pxxxxxQ==

---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: dns-issuer-rfc2136-live
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: "teamsmiley@gmail.com"
    privateKeySecretRef:
      name: dns-issuer-rfc2136-live
    solvers:
      - dns01:
          rfc2136:
            nameserver: 172.21.1.20:53
            tsigKeyName: teamsmiley-dev-secret
            tsigAlgorithm: HMACSHA512
            tsigSecretSecretRef:
              name: tsig-secret
              key: tsig-secret-key

```

## Ingress에서 사용

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: teamsmiley-dev
  annotations:
    cert-manager.io/cluster-issuer: dns-issuer-rfc2136-live #주의
spec:
  ingressClassName: nginx-internal
  rules:
    - host: www.teamsmiley.dev
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: www
                port:
                  number: 80
  tls:
    - hosts:
        - www.teamsmiley.dev
      secretName: www-tls
```

잘 생성되는지 확인한다.


<https://cert-manager.io/docs/configuration/acme/dns01/rfc2136/#configuration-step-1-set-up-your-dns-server-for-secure-dynamic-updates>
