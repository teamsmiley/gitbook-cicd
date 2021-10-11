# TIP

쿠버네티스를 사용중에 나오는 tip들을 정리해보려고 한다.

## 완료된 리소스를 위한 TTL 컨트롤러

[https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/ttlafterfinished/)

## root가 아닌 사용자로 컨테이너 실행하기

```yml
apiVersion: v1
kind: Pod
metadata:
  name: securityContext
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
```

도커 이미지에서도 지정이 가능하나 이렇게 쿠버네티스에서 변경도 가능하다.

1000번은 리눅스에서 처음 유저가 가지는 번호이므로 1000번 이상을 사용하는게 좋다.

쿠버네티스가 도커의 모든 유저 설정을 덮어쓴다.

각 컨테이너마다 다른 uid를 추천한다.

두개의 컨테이너가 동일한 데이터에 접근하면 uid는 같아야한다.

## root container 차단하기

쿠버네티스가 container가 root로 실행되는것을 방지해주는 옵션을 제공

```yml
apiVersion: v1
kind: Pod
# ...
spec:
  securityContext:
    runAsNonRoot: true
```

root로 실행되는것을 방지해준다.

`CreateContainerConfigError: container 'root' is not allowed to run as root` 발생

## readOnlyRootFile

```yml
apiVersion: v1
kind: Pod
# ...
spec:
  securityContext:
    readOnlyRootFileSystem: true
```

컨테이너에 쓰기 권한이 필요하지 않으면 옵션을 사용하자.

## 권한 상승 비활성화

```yml
apiVersion: v1
kind: Pod
# ...
spec:
  securityContext:
    allowPrivilegeEscalation: false
```

리눅스 바이너리는 이를 실행한 사용자와 같은 권한으로 실행된다. 그러나 예외는 있다.

seuid 매커니증을 사용하면 일시적으로 바이너리를 소유한 사용자(일반적으로 root)의 권한으로 실행할수 있다.

컨테이너를 일반 사용자로 실행하더라도 컨테이너가 setuid 바이너리를 가지고 있는 경우에 컨테이너에서 root 권한을 얻을수 잇으므로 잠재적인 문제가 된다. 이를 방지하기위해 `allowPrivilegeEscalation` 를 사용한다.

## 컨테이너 단위로 권한 설정

위 설정들은 pod가 아니라 컨테이너 수준으로 올라갈수도 있다.

## 클러스터 전체에 설정

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

## 이미지 태그

사용시 꼭 태그 넘버를 사용 아무것도 안쓰면 latest가 자동으로 붙는다.

latest 를 사용하지 말자. 계속 버전이 바뀌므로 문제가 된다.

사실 sha 태그도 중복이 된다. 가능하면 container digest를 사용하자.

digest는 유일하다.

`docker image ls --digests`

![](../.gitbook/assets/2021-10-08-08-03-01.png)

환경변수의 최대값은 32KiB로 제한

## feature-gates

[https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/](https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/)

기능 게이트는 쿠버네티스 기능을 설명하는 일련의 키=값 쌍이다. 각 쿠버네티스 컴포넌트에서 --feature-gates 커맨드 라인 플래그를 사용하여 이러한 기능을 켜거나 끌 수 있다.
