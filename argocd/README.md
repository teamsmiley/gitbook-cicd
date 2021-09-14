# Argocd

best practices 를 읽어보고 정리

## 구성과 소스코드 저장소를 분리해서 사용

1. 단순히 복제 수를 늘리려고 소스코드 빌드를 트리거 하고 싶지 않다.
1. 별도의 저장소와 별도의 권한이 필요
1. 소스코드 수정후 git commit의 무한 트리거가 발생될수 있다. 구성변경사항을 푸히할 별도의 저장도가 있으면 이런일이 발생하지 않는다.

## Leaving Room For Imperativeness

Horizontal Pod Autoscaler 에서 배포 복제본의 수를 관리 하려면 replicasGit 에서 추적하고 싶지 않을 것 입니다.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  # do not include replicas in the manifests if you want replicas to be controlled by HPA
  # replicas: 1
  template:
    spec:
      containers:
        - image: nginx:1.7.9
          name: nginx
          ports:
            - containerPort: 80
```

## Ensuring Manifests At Git Revisions Are Truly Immutable

kustomization.yaml 에서 다음처럼 사용하면

```yml
bases:
  - github.com/argoproj/argo-cd//manifests/cluster-install
```

upstream에서 업데이트가 되어서 업데이트가 되버리는경우가 있다. 꼭 버전을 사용하자.

```yml
bases:
  - github.com/argoproj/argo-cd//manifests/cluster-install?ref=v0.11.1
```
