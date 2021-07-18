# Prometheus

서버 포드 서비스 등을 모니터링 하기 위한 툴



쿠버네티스클러스터당 각자 1개씩 추천 . 왜냐면 서비스 디스커버리랑 내부 pod가 통신이 되야하니까.

argocd repo에  submodule로 등록을 햇다.



```text
git submodule add https://github.com/prometheus-operator/kube-prometheus.git core/kube-prometheus

cd core/kube-prometheus

git checkout tags/v0.8.0

cd ../../

git add --all 
git commit -m "kube-prometheus: v0.8.0" 
git push
```



