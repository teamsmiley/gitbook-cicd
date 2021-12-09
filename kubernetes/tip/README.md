# TIP

쿠버네티스를 사용중에 나오는 tip들을 정리해보려고 한다.

## 클러스터 전체에 권한 설정

PodSecurityPolicy를 사용하면 되었으나 쿠버네티스 v1.21부터 더이상 사용되지 않으며, v1.25에서 제거된다.

사용 중단에 대한 상세 사항은 [여기](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/) 참조

Kubernetes Enhancement Proposal 라는 이름으로 22 에서 알파 릴리즈를 한다고 합니다.

조금 관심가지고 확인해봐야할거같습니다.

## feature-gates

[https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/](https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/)

기능 게이트는 쿠버네티스 기능을 설명하는 일련의 키=값 쌍이다. 각 쿠버네티스 컴포넌트에서 --feature-gates 커맨드 라인 플래그를 사용하여 이러한 기능을 켜거나 끌 수 있다.

## 완료된 리소스를 위한 TTL 컨트롤러

[https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/)

## kubectl port-forward

```sh
kubectl port-forward svc/argocd-server -n argocd 8000:443
```

기본적으로 127.0.0.1에만 포트를 오픈한다 모든 아이피에 오픈하고 싶으면 다음처럼 하자.

```sh
kubectl port-forward svc/argocd-server -n argocd 8000:443 --address='0.0.0.0'
```
