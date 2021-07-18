# git submodule

git submodule을 지원한다. 



git submodule add [https://github.com/prometheus-operator/kube-prometheus.git](https://github.com/prometheus-operator/kube-prometheus.git) core/kube-prometheus

cd core/kube-prometheus

git checkout tags/v0.8.0

cd ../../

git add --all && git commit -m "kube-prometheus: v0.8.0" && git push

```text
### 사용

```sh
git submodule update --init --recursive
```

