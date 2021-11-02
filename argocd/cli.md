# CLI

## install argocd cli

[https://argo-cd.readthedocs.io/en/stable/cli_installation/](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

```bash
# macos
brew install argocd

# linux latest
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

# linux version
VERSION=<TAG> # Select desired TAG from https://github.com/argoproj/argo-cd/releases
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

[https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/)

## port forwarding

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

접속 확인

[https://localhost:8080](https://localhost:8080)

## get password

```sh
k -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d && echo
```

## login

```bash
argocd login localhost:8080
>admin
```

## password 변경

```bash
# argocd account update-password --account <new-username> --new-password <new-password>
argocd account update-password
argocd account update-password --account admin --new-password xxxx
```

## add user

create configmap

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
data:
  # add an additional local user with apiKey and login capabilities
  #   apiKey - allows generating API keys
  #   login - allows to login using UI
  accounts.dev: login
  # disables user. User is enabled by default
  accounts.dev.enabled: 'false'
  # add an additional local user with apiKey and login capabilities
  # admin
  accounts.admin: apiKey #token을 만들기위해서 넣어줌.
```

```yml
k apply -f argocd-cm.yml
```

## create token

```sh
argocd account generate-token --account dev
```

--account가 없으면 현재 로그인된 유저

어드민의 경우 위의 예제처럼 추가로 넣어줘야한다. 그리고 커맨드로 생성시 에러가 나는경우도 있다. 이때는 웹에서 생성하면된다.

## role 추가

유저를 추가해도 권한을 안주면 아무것도 보이지 않는다.

업데이트해보자.

```yml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:dev, applications, get, */*, allow
    p, role:dev, applications, sync, */*, allow
    p, role:dev, repositories, get, *, allow
    g, dev, role:dev

# https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/
```

```sh
k apply -f argocd-rbac-cm.yml
```

dev라는 유저를 권한을 추가했고 sync가 가능하게 햇다.

## add repo

```bash
argocd repo add git@github.com:YOUR/argocd.git \
--ssh-private-key-path ~/.ssh/argocd
```

## add app

```bash
# Create a directory app
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-namespace default \
  --dest-server https://kubernetes.default.svc \
  --directory-recurse

# Create a Kustomize app
argocd app create kustomize-guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path kustomize-guestbook \
  --dest-namespace default \
  --dest-server https://kubernetes.default.svc \
  --kustomize-image gcr.io/heptio-images/ks-guestbook-demo:0.1
```

## get app

```bash
argocd app get APPNAME
```

## app sync

```bash
NAME=default
argocd app sync ${NAME} --prune --force
argocd app wait ${NAME} --timeout 1200
```

## app delete

```sh
argocd app delete homesecurity-hms
argocd app delete homesecurity-hms --cascade=false
```

## how to use token

```bash
export ARGOCD_SERVER=argocd.mycompany.com
export ARGOCD_AUTH_TOKEN=<JWT token generated from project>
argocd app sync guestbook
argocd app wait guestbook
```

## refernce

{% embed url="https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/" %}
argocd cli
{% endembed %}
