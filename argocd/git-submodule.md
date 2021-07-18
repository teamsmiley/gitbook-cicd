# git submodule

git submodule을 지원한다.

prometheus를 설치해보자.

[https://github.com/prometheus-operator/kube-prometheus](https://github.com/prometheus-operator/kube-prometheus)

```bash
git submodule add https://github.com/prometheus-operator/kube-prometheus.git core/kube-prometheus

cd core/kube-prometheus

git checkout tags/v0.8.0

cd ../../

git add --all && git commit -m "kube-prometheus: v0.8.0" && git push

k apply -f add-apps/core/kube-prometheus.yaml #app추가해서 사용하면된다.
```

metrics-server가 있으면 에러가 나서 기존 metrics-server지워주었다.
