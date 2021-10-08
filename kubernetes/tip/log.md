# Kubernetes에서 로그

### 기본 사용법

기본적으로 kube에서는 다음처럼 로그를 보면된다.

```bash
kubectl logs -f  XXXX
```

이러면 로그를 계속 볼수 있다. 그런데 너무 로그가 많으면 오래 걸린다. 특별히 시간을 줄수 있다.

```bash
kubectl logs -f xxxx  --since=5m
```

이제 5분전 로그부터 볼수 있다.

### deployment된 모든 pod의 로그 확인

replica 세팅을 하면 같은 docker가 여러개 올라간다. 이런데 리퀘스트를 던지면 어떤 pod로 들어간지 알수가 없어서 불편하다.

deployment된 모든 pod의 로그를 보고 싶다.

```bash
kubectl logs -f deployment/mobile-php --all-containers=true --since=5m
```

이러면 이제 mobile-php 로 생성된 모든 pod의 로그를 한꺼번에 보여준다.

- grep 으로 필터도 가능하겠다.
- k9s에서는 deployment를 선택한후 l을 눌러서 로그를 보면 전체 pod의 로그를 볼수 있다.

## service log

서비스에 포함된 모든 pod의 로그를 보여준다.

```sh
k logs -f svc/argocd-repo-server
```
