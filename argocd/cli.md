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

## login

```bash
argocd login localhost:8080
```

## password 변경

```bash
argocd account update-password
```

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

## app sync (adv)

```bash
export ARGOCD_SERVER=argocd.mycompany.com
export ARGOCD_AUTH_TOKEN=<JWT token generated from project>
curl -sSL -o /usr/local/bin/argocd https://${ARGOCD_SERVER}/download/argocd-linux-amd64
argocd app sync guestbook
argocd app wait guestbook
```

## refernce

{% embed url="https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/" %}
argocd cli
{% endembed %}
