# PMM

## pmm server

install

```bash
helm repo add percona https://percona-charts.storage.googleapis.com
helm repo update
NS=pxc-mysql
helm install monitoring pmm/pmm-server -n $NS --set platform=kubernetes --set "credentials.password=your_password"
```

ingress 설정

cert-manager가 설정이 미리 되있어서 ssl까지 만들면서 진행

백앤드에 ssl로 통신하는것 중요

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pmm
  namespace: pxc-mysql
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: 'dns-issuer-aws-live'
    nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'
    nginx.ingress.kubernetes.io/backend-protocol: 'HTTPS' # 이부분 꼭 확인
spec:
  tls:
    - hosts:
        - 'pmm.c3.yourdomain.com'
      secretName: pmm-tls
  rules:
    - host: pmm.c3.yourdomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: pxc-pmm-service
                port:
                  number: 443
```

사이트에 접속해보면 grafana가 보인다. 로그인하면 된다.

## pmm-client

{% code title="cr.yaml" %}

```yml
pmm:
  enabled: true
  image: percona/pmm-client:2.18.0
  serverHost: pxc-pmm-service # pmm-server에서의 서비스 명
  serverUser: admin # 확인
```

{% endcode %}

```bash
k apply -f cr.yaml
```

자동으로 서버를 찾아서 자기 스스로를 등록한다.

## pmm 확인

pmm.c3.yourdomain.com 으로 들어가서 확인해보면 많은 데이터를 볼수 있다.

다 구성되고 나면 pmm 에 접속해보면 클러스터 상태가 보인다.

![](../.gitbook/assets/2021-08-19-06-22-40.png)

![](../.gitbook/assets/2021-08-19-06-27-22.png)

![](../.gitbook/assets/2021-08-19-06-38-25.png)

alert manager를 설정하면 슬랙으로 에러를 받을수 있다.
