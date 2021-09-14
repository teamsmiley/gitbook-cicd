# Install

## create git repo

github/gitlab에 argocd용 repo를 하나 만들자.

```sh
git clone argocd
cd argocd
```

클론을 받아서 사용할 준비를 하자.

## install argocd

```sh
kubectl create namespace argocd

curl -o argocd_install_v2.0.3.yaml https://raw.githubusercontent.com/argoproj/argo-cd/v2.0.3/manifests/install.yaml

kubectl apply -n argocd -f argocd_install_v2.0.3.yaml

kubectl get all --all-namespaces
```

## 접속 확인

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

{% embed url="http://localhost:8080" caption="" %}

![](../.gitbook/assets/argocd-install-01.png)

## 비번 알아내기

```sh
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d && echo
```

admin으로 접속하면 된다.
