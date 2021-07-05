# CLI

## install argocd cli

[https://argo-cd.readthedocs.io/en/stable/cli_installation/](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

```bash
brew install argocd
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
