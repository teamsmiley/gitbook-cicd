# TIP

쿠버네티스를 사용중에 나오는 tip들을 정리해보려고 한다.

이 페이지에 쓰다가 많아지면 subpage로 이동하는중

## feature-gates

<https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/>

기능 게이트는 쿠버네티스 기능을 설명하는 일련의 키=값 쌍이다. 각 쿠버네티스 컴포넌트에서 --feature-gates 커맨드 라인 플래그를 사용하여 이러한 기능을 켜거나 끌 수 있다.

## 완료된 리소스를 위한 TTL 컨트롤러

<https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/>
