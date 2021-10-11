# TIP

쿠버네티스를 사용중에 나오는 tip들을 정리해보려고 한다.

## 클러스터 전체에 권한 설정

PodSecurityPolicy를 사용하면 되었으나 쿠버네티스 v1.21부터 더이상 사용되지 않으며, v1.25에서 제거된다.

사용 중단에 대한 상세 사항은 [여기](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/) 참조

Kubernetes Enhancement Proposal 라는 이름으로 22 에서 알파 릴리즈를 한다고 합니다.

조금 관심가지고 확인해봐야할거같습니다.

## SOPS로 시크릿 암호화 하기

SOPS: secrets operations

yml/json 등을 암호화 하고 복호화 하는 기능

<https://github.com/mozilla/sops>

source에서 암호화 하고 배포시 복호화해서 사용하는 방식

gitops에서는 안되겟음? ..argocd repo가 복호화 된 평문을 가지고 있으므로.

## feature-gates

[https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/](https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/)

기능 게이트는 쿠버네티스 기능을 설명하는 일련의 키=값 쌍이다. 각 쿠버네티스 컴포넌트에서 --feature-gates 커맨드 라인 플래그를 사용하여 이러한 기능을 켜거나 끌 수 있다.

## 완료된 리소스를 위한 TTL 컨트롤러

[https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/)
