# pod

## 이미지 태그

사용시 꼭 태그 넘버를 사용 아무것도 안쓰면 latest가 자동으로 붙는다.

latest 를 사용하지 말자. 계속 버전이 바뀌므로 문제가 된다.

사실 sha 태그도 중복이 된다. 가능하면 container digest를 사용하자.

digest는 유일하다.

`docker image ls --digests`

![](../.gitbook/assets/2021-10-08-08-03-01.png)

환경변수의 최대값은 32KiB로 제한

## delete completed pod

```bash
kubectl delete pod --field-selector=status.phase==Succeeded
```

## env를 configmap으로 이용하기

configmap 이 있는 상황에서 pod에서 env값으로 configmap을 이용하기

```text
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ['/bin/sh', '-c', 'env']
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
```

## envFrom를 configmap으로 이용하기

```text
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ['/bin/sh', '-c', 'env']
      envFrom:
        - configMapRef:
            name: special-config
  restartPolicy: Never
```

이렇게 하면 configmap에 있던 모든 내용이 env값으로 변환된다.

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
