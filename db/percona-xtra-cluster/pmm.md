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

```text
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

![](https://github.com/teamsmiley/gitbook-cicd/tree/c4b68f9c45e45a9eb17b11a061898c948c5d845c/db/.gitbook/assets/2021-08-19-06-22-40.png)

![](https://github.com/teamsmiley/gitbook-cicd/tree/c4b68f9c45e45a9eb17b11a061898c948c5d845c/db/.gitbook/assets/2021-08-19-06-27-22.png)

![](https://github.com/teamsmiley/gitbook-cicd/tree/c4b68f9c45e45a9eb17b11a061898c948c5d845c/db/.gitbook/assets/2021-08-19-06-38-25.png)

alert manager를 설정하면 슬랙으로 에러를 받을수 있다.

## subchart 로 argocd에서 설정

argocd 에서 subchart를 사용 해야 gitops가 된다.

subchart로 만들자.

```sh
mkdir pmm
cd pmm

touch Chart.yaml
```

```yml
apiVersion: v2
name: pmm-subchart
type: application
version: 1.0.0
appVersion: '1.0.0'
dependencies:
  - name: pmm-server
    version: 2.18.0
    repository: https://percona-charts.storage.googleapis.com
```

vi values.yaml

```yml
pmm-server:
  ## percona image version
  ## ref: https://hub.docker.com/r/library/percona/tags/
  ##
  imageRepo: 'percona/pmm-server'
  imageTag: '2.18.0'

  ## A choice between "kubernetes" and "openshift"
  platform: 'kubernetes'

  ## Specify an imagePullPolicy (Required)
  ## It's recommended to change this to 'Always' if the image tag is 'latest'
  ## ref: http://kubernetes.io/docs/user-guide/images/#updating-images
  ##
  imagePullPolicy: Always
  scc: null
  sa: null
  ## Persist data to a persitent volume
  persistence:
    enabled: true
    ## percona data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    # storageClass: "-"
    accessMode: ReadWriteOnce
    size: 30Gi

  ## set credentials
  credentials:
    password: 'admin'

  ## set metric collection settings
  metric:
    resolution: 1s
    retention: 720h
  queries:
    retention: 8

  ## Configure resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  ##
  resources:
    requests:
      memory: 1Gi
      cpu: 0.5

  supresshttp2: true
  service:
    type: LoadBalancer
    port: 443
    loadBalancerIP: ''

  ## Mount prometheus scrape config https://www.percona.com/blog/2020/03/23/extending-pmm-prometheus-configuration/
  prometheus:
    configMap:
      name: ''

  ## Kubernetes Ingress https://kubernetes.io/docs/concepts/services-networking/ingress
  ingress:
    enabled: false
    annotations:
      {}
      # kubernetes.io/ingress.class: nginx
      # kubernetes.io/tls-acme: "true"
    path: /
    pathType: null
    host: monitoring-service.example.local
    rules: []
    tls: []
    #  - secretName: pmm-server-tls
    #    hosts:
    #      - monitoring-service.example.local
    labels: {}
```

vi add-pmm-server.yaml

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: pxc-pmm-server
  namespace: argocd
spec:
  destination:
    name: ''
    namespace: pxc-mysql
    server: 'https://kubernetes.default.svc'
  source:
    path: apps/pxc-pmm-server
    repoURL: 'git@github.com:teamsmiley/argocd-c3.git'
    targetRevision: HEAD
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

`k apply -f add-pmm-server.yaml`


잘 적용된다.
