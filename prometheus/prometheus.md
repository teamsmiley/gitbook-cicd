# Prometheus

서버 포드 서비스 등을 모니터링 하기 위한 툴

쿠버네티스클러스터당 각자 1개씩 추천 . 왜냐면 서비스 디스커버리랑 내부 pod가 통신이 되야하니까.

argocd repo에 submodule로 등록을 햇다.

```bash
git submodule add https://github.com/prometheus-operator/kube-prometheus.git core/kube-prometheus

cd core/kube-prometheus

git checkout tags/v0.8.0

cd ../../

git add --all
git commit -m "kube-prometheus: v0.8.0"
git push
```

app등록하면 디플로이가 되는 것을 볼수 있다.

```bash
kubectl -n monitoring port-forward svc/prometheus-k8s 9090
```

```bash
kubectl -n monitoring port-forward svc/alertmanager-main 9093
```

```bash
kubectl -n monitoring port-forward svc/grafana 3000
```

## prometheus

[http://localhost:9090](http://localhost:9090)

## alertmanager

[http://localhost:9093](http://localhost:9093)

## grafana

[http://localhost:3000/login](http://localhost:3000/login) admin/admin

로그인후 dashboard &gt; manage &gt; default 에 가면 미리 설치된 대시보드를 볼수 있다.

## ingress-nginx 모니터링 하기

https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md

여기를 보면 serviceMonitor 볼수 있다 이걸 만들어 주면 쿠버네티스에서 모니터링이 가능하다.

그런데 ingress-nginx는 벌서 가지고 있다 활성화면 해주면 된다.

```yml
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

## serviceMonitor 추가하기

```yml
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
