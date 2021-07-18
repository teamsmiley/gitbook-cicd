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
