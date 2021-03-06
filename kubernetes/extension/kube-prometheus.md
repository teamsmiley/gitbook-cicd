# kube-prometheus

## prometheus

서버의 메트릭을 모니터링을 하기 위한 툴

<https://github.com/prometheus>

보통은 그라파나와 같이 사용하여 서버 메트릭을 모니터링한다.

## what is kube-prometheus

<https://github.com/prometheus-operator/kube-prometheus>

k8s에서 메트릭을 모니터링 하기 위한 툴

쿠버네티스클러스터당 각자 1개씩 추천 . 왜냐면 서비스 디스커버리랑 내부 pod가 통신이 되야한다. 외부도 모니터링이 가능은 하나 구지 그럴필요 없어 보인다. 

이건 k8s 클러스터만 모니터링 하는것으로 사용하자. 

기본적으로 메모리에만 저장되는듯 보인다. 그래서 상태를 저장하려면 추가 작업이 필요하다.

## 클러스터에 설치하기

jb라는 유틸리티가 필요하다.

```sh
brew install jsonnet-bundler # for jb command
```

```sh
cd ~/Desktop
git clone https://github.com/prometheus-operator/kube-prometheus.git 
```

나는 argocd를 k8s 설정으로 사용하기 때문에 argocd git repo에 넣고 싶었다. 

git에 submodule을 이용하자.

초기 등록은 이렇게 한다.

```sh
cd ~/Desktop/k8s-config # argocd repo

git submodule add https://github.com/prometheus-operator/kube-prometheus.git core/kube-prometheus
cd core/kube-prometheus
git branch tags/v0.9.0
git add --all && git commit -am "add submodule kube-prometheus" && git push
```

argocd repo를 다운받을때는 다음처럼 사용

```sh
git clone --recursive xxxx 
```

미리 받아둔 repo면

```sh
cd xxxx
git submodule update --init --recursive
```

빌드하자.

```sh
cd core/kube-prometheus
jb install # vendor 폴더가 생긴다.
docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh example.jsonnet
```

manifest폴더가 생긴다. 이걸 argocd repo에 넣고 argocd에서 app등록하면 디플로이가 되는 것을 볼수 있다.

example.jsonnet을 각자의 상황에 맞게 이름을 바꿔서 사용한다.

이파일을 수정하면  수정된 manifest가 생성이 된다.


## 확인

포트 포워딩으로 확인할수 있다.

```sh
kubectl -n monitoring port-forward svc/prometheus-k8s 9090
kubectl -n monitoring port-forward svc/alertmanager-main 9093
kubectl -n monitoring port-forward svc/grafana 3000
```

### prometheus

[http://localhost:9090](http://localhost:9090)

### alertmanager

[http://localhost:9093](http://localhost:9093)

### grafana

[http://localhost:3000/login](http://localhost:3000/login) 


## customize manifest

example.jsonnet을 수정 해서 manifest 수정

grafana/prometheus/alertmanager svc가 현재는 clusterip 인데 node port로 변경해보자.

`kube-prometheus/addons/node-ports.libsonnet` 이부분만 주석 해제 해주면된다.

```text
local kp =
  (import 'kube-prometheus/main.libsonnet') +
  (import 'kube-prometheus/addons/node-ports.libsonnet')
  {
```

다시 빌드하고 커밋하면된다.

```sh
docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh example.jsonnet
```

커밋후 svc를 확인해보면 서비스 타입이 노드포트로 변경되는걸 알수 있다.

이제 다시 지워고 다시 빌드 커밋 푸시 하면 원래대로 돌아오는것을 알수 있다.

## ingress를 통해서 public domain으로 접근가능하게

[https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/exposing-prometheus-alertmanager-grafana-ingress.md](https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/exposing-prometheus-alertmanager-grafana-ingress.md)

참고해서 봐도 되고 다음 코드도 같이 보면 좋다.

[https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/ingress.jsonnet](https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/ingress.jsonnet)

example.jsonnet을 위 파일처럼 수정후 domain을 변경해주면된다.

### basic auth 사용

auth라는 파일을 참조하는것을 알수 있다.

이걸 만들기 위해서는 다음과 같이 한다.

```sh
USER=admin; PASSWORD=XXXXX; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" > auth
```

auth라는 파일이 생겼다. 내용을 복사하여 example.jsonnet파일과 같은 디렉토리에 복사해서 넣어준다.

도메인으로 접근하면 basic login화면이 나오고 생성해준 id 비번을 넣으면 로그인이 된다.

하고보면 grafana는 id/password를 두번 넣어야하는 문제가 생기는데?

grafana는 기본인증에서 빼도 될듯 보인다. grafana를 수정햇다. ingress라는 함수를 안쓰고 직접 넣어준다.

```javascript
grafana: {
  apiVersion: 'networking.k8s.io/v1',
  kind: 'Ingress',
  metadata: {
    name: 'grafana',
    namespace: $.values.common.namespace,
  },
  spec: {
    rules: [{
      host: 'grafana.c3',
      http: {
        paths: [{
          path: '/',
          pathType: 'Prefix',
          backend: {
            service: {
              name: 'grafana',
              port: {
                name: 'http',
              },
            },
          },
        }],
      },
    }],
  },
},
```

## etcd 모니터링

```sh
brew install cfssl # install cfssl

cd core/kube-prometheus

scp master1-eqix-sv5:/etc/ssl/etcd/ssl/ca.pem etcd/
scp master1-eqix-sv5:/etc/ssl/etcd/ssl/ca-key.pem etcd/

chmod 755 etcd/*.pem

vi etcd/client.json

cat <<EOF > etcd/client.json
{
  "CN": "etcd-ca",
  "hosts": [""],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{}]
}
EOF
```

```sh
# Generate client certificate
cfssl gencert -ca ca.pem -ca-key ca-key.pem client.json | cfssljson -bare etcd-client
```

jsonnet 설정

[https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/etcd.jsonnet](https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/etcd.jsonnet)

여기 참고하면된다.

아이피는 사용하는 아이피 전부 넣어주면되고 서버이름은 빈칸으로 해도 된다. insecureSkipVerify 는 false로

```text
etcd+: {
        ips: ['172.16.3.11', '172.16.3.12', '172.16.3.13'],
        clientCA: importstr 'etcd/ca.pem',
        clientKey: importstr 'etcd/etcd-client-key.pem',
        clientCert: importstr 'etcd/etcd-client.pem',
        //serverName: 'etcd.kube-system.svc.cluster.local',
        serverName: '',

        insecureSkipVerify: true,
      },
```

manifest를 업데이트하자.

```sh
docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh c4.jsonnet
```

prometheus 웹에 가서 etcd_cluster_version 으로 검색해서 나오면 확인된다.

![](./images/2022-07-22-23-17-30.png)

## instance가 하나의 노드에 2개뜨는걸 방지

현재 alertmanager-main이 node05에 두개가 떠 있다. 이걸 다른노드에서 띄워보자.

[https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/anti-affinity.jsonnet](https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/anti-affinity.jsonnet)

참고해서 주석만 한줄 풀어줬다.

서로 다른 노드에 배포되는것을 확인했다.

## alert

슬랙으로 alert를 받고 싶다.

일단 슬랙채널을 만들어보자.

![](../.gitbook/assets/2021-07-19-07-32-51.png)

웹 후크 관련 설정을 한다. [https://api.slack.com/messaging/webhooks](https://api.slack.com/messaging/webhooks)

실제 메세지가 가는지 테스트 한다.

[https://prometheus.io/docs/alerting/latest/notification_examples/](https://prometheus.io/docs/alerting/latest/notification_examples/)

```sh
mkdir alertmanager

cat <<EOF > alertmanager/config.yml
global:
  resolve_timeout: 1m
  slack_api_url: 'https://hooks.slack.com/services/T/B01P/ddd'
route:
  receiver: 'slack-notifications'
receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#kube'
        send_resolved: true
        icon_url: 'https://avatars3.githubusercontent.com/u/3380462'
        title: '[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] Monitoring Event Notification'
        text: |-
          {{ range .Alerts }}
            *Alert:* {{ .Annotations.summary }} - `{{ .Labels.severity }}`
            *Description:* {{ .Annotations.description }}
            *Graph:* <{{ .GeneratorURL }}|:chart_with_upwards_trend:>
            *Details:*
            {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
            {{ end }}
          {{ end }}
EOF
```

웹 후크 url을 적어주고 나머지는 잘 수정해서 보내준다.

jsonnet 파일을 수정한다.

```text
values+:: {
     ...

      // Change Alertmanager configuration
      alertmanager+: {
        config: importstr 'alertmanager/config.yaml',
      },
```

컴파일 하고 올려보자.

문제가 생기면 슬랙으로 알림이 잘 온다.

## KubeSchedulerDown-alert

KubeSchedulerDown 알림이 계속온다.

v0.8에서 조금 이상해진거같음.

```text
values+:: {

      kubePrometheus+: {
        platform: 'kubeadm',
      },
```

이걸 추가하면 에러가 없어진다고 하는데 ..해보자.

없어진다.

### CPUThrottlingHigh-alert

CPUThrottlingHigh가 계속 알림으로 온다. node-exporter가 cpu가 높다는것이다.

내용을 확인해보자. manifest파일을 확인해보니 다음과 같다.

```text
- alert: CPUThrottlingHigh
      annotations:
        description: '{{ $value | humanizePercentage }} throttling of CPU in namespace {{ $labels.namespace }} for container {{ $labels.container }} in pod {{ $labels.pod }}.'
        runbook_url: https://github.com/prometheus-operator/kube-prometheus/wiki/cputhrottlinghigh
        summary: Processes experience elevated CPU throttling.
      expr: |
        sum(increase(container_cpu_cfs_throttled_periods_total{container!="", }[5m])) by (container, pod, namespace)
          /
        sum(increase(container_cpu_cfs_periods_total{}[5m])) by (container, pod, namespace)
          > ( 25 / 100 )
      for: 15m
      labels:
        severity: info
```

값이 25이상이면 보내게 되있다. 해결방안을 고민해보자.

1. 25이상이 무리가 없다고 판단되면 예를들어 50%까지는 알림을 보고싶지 않다고 하면 25를 50으로 바꾸면 되지 낳을가?
2. 해당 pod의 resource를 추가해 줘야 하지 않을가?

node-exporter-daemonset.yaml 에서 다음 부분을 수정해야 한다.

```text
resources:
  limits:
    cpu: 250m
    memory: 180Mi
  requests:
    cpu: 102m
    memory: 180Mi
```

일단 request를 cpu 250m으로 해보고 알림이 오는지 확인해보자.

일단 기존보다는 %가 내려간것을 알수 있다.

![](../.gitbook/assets/2021-07-20-10-06-13.png)

여전히 25가 넘어가면 알림이 발생 50으로 변경해서 테스트

알림이 줄어들었다.

이제 컴파일시 저 숫자들을 변경해줘야하는데..

```text
values+:: {
  ...
  kubernetesControlPlane+: {
    mixin+: {
      _config+: {
        cpuThrottlingPercent: 60,
      },
    },
  },

}
```

이렇게 하고 컴파일 푸시하면 된다.

[https://github.com/prometheus-operator/kube-prometheus/issues/1165](https://github.com/prometheus-operator/kube-prometheus/issues/1165)

## api error burn rate

이 에러가나서 확인해봣더니 노드에서 다음 에러가 나온다.

```text
Search Line limits were exceeded, some search paths have been omitted, the applied search line
```

`/etc/resolve.conf`에 보면 여러개의 search에 항목이 있엇다. 전부 지워주니 에러도 없어졌고 알람도 없어졋다.

## grafana customize

DataSource prom/loki 를 기본추가, id/pass추가

// datasource가 하나도 없으면 prometheus datasource는 자동으로 넣어준다. 그런데 아래처럼 loki를 넣어버리면 prometheus datasource가 자동으로 생성이 안되는듯 보인다. 그래서 아래처럼 따로 추가해주었다.

아래 url은 같은 namespace에서는 서비스이름으로 접속이 가능하다. 다르면 servicename.namespace.svc.cluster.local 이런식으로 접속이 가능하다. 또는 servicename.namespace.svc

```javascript
grafana+:: {
  // Add DataSource
  datasources+: [
    {
      name: 'prometheus',
      type: 'prometheus',
      access: 'proxy',
      orgId: 1,
      url: 'http://prometheus-k8s:9090',
      editable: false,
    },
    {
      name: 'loki',
      type: 'loki',
      access: 'proxy',
      orgId: 2,
      url: 'http://core-loki-stack:3100',
      editable: false,
    },
  ],

  config+: {
    sections+: {
      'security': {
        admin_user: 'admin',
        admin_password: 'yourpassword'
      },

      server+: {
        root_url: 'https://grafana.yourdomain/',
      },
    },
  },
},
```

## ingress-nginx 모니터링 하기

[https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md)

여기를 보면 serviceMonitor 볼수 있다 이걸 만들어 주면 쿠버네티스에서 모니터링이 가능하다.

그런데 ingress-nginx는 벌서 가지고 있다 활성화면 해주면 된다.

```text
- name: controller.metrics.enabled
  value: 'true'
- name: controller.metrics.serviceMonitor.enabled
  value: 'true'
- name: controller.metrics.serviceMonitor.namespace
  value: ingress-nginx
```

이러면 된다. 프로메테우스에서 데이터를 모아서 그라파나로 보여준다.

그라파나 템플릿 번호는 9614번이 좋아보인다.

잘보인다.

여기서 알아야할점은 serviceMonitor 추가하면 프로메테우스에서 모니터링이 가능하다.

```text
controller.metrics.enabled=true
controller.metrics.serviceMonitor.enabled=true
controller.metrics.serviceMonitor.additionalLabels.release="prometheus" 
```

타켓에 추가된걸 알수가 있다.

![](./images/2022-07-23-00-02-05.png)


![](./images/2022-07-22-23-59-43.png)


## serviceMonitor 추가하기

```text
kind: Service
apiVersion: v1
metadata:
  name: example-app
  labels:
    app: example-app
spec:
  selector:
    app: example-app
  ports:
    - name: web
      port: 8080

---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-app
  labels:
    team: frontend
spec:
  selector:
    matchLabels:
      app: example-app
  endpoints:
    - port: web
```
