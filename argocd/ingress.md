# ingress

ERR\_TOO\_MANY\_REDIRECTS

argocd를 insecure하게 실행하지 않았기 때문이다. 

```text
spec:
  containers:
  - command:
    - argocd-server
    - --staticassets
    - /shared/app
    - --insecure

```

이렇게 insecure를 추가하면된다.


