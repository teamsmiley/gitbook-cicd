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

```bash
cd ~/Desktop

git clone https://github.com/prometheus-operator/kube-prometheus.git 
```

나는 argocd를 k8s 설정으로 사용하기 때문에 argocd git repo에 넣고 싶었다. 

git에 submodule을 이용하자. 

초기 등록은 이렇게 한다.

```sh
cd ~/Desktop/k8s-config # argocd repo

git submodule add https://github.com/prometheus-operator/kube-prometheus.git core/kube-prometheus
git add --all && git commit -am "add submodule kube-prometheus" && git push
```

argocd repo를 다운받을때는 다음처럼 사용

```sh
git clone xxxx
git submodule update
# 한꺼번에 클론할수도 있다. git clone xxx --recursive
```


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

