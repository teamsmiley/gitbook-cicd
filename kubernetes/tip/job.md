# job

## delete completed job

```sh
kubectl delete jobs --field-selector status.successful=1
```

## delete fail job or long running job

```sh
kubectl delete jobs --field-selector status.successful=0
```

## TTLAfterFinished

이 기능이 1.21 버전부터는 기본으로 포함이 되었다.

현재까지는 job에만 적용이 되나 나중에 확장될 예정 1.21에서 beta

[https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/](https://kubernetes.io/ko/docs/reference/command-line-tools-reference/feature-gates/)

job에 .spec.ttlSecondsAfterFinished 를 설정하면 그시간 이후로 자동으로 삭제된다.

percona xtradb cluster operator에서는 .spec.ttlSecondsAfterFinished 을 포함하지 않는듯 보인다.

이게 지원이 될때까지 수동으로 지워야할듯
