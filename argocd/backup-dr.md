# Backup / DR

https://argoproj.github.io/argo-cd/operator-manual/disaster_recovery/

```bash
argocd login argocd.rendercore.com

argocd version | grep server

> v2.0.3+8d2b13d

export VERSION=v2.0.3

#Export to a backup:
docker run -v ~/.kube/aws-rendercore:/home/argocd/.kube/config --rm argoproj/argocd:$VERSION argocd-util export > backup.yaml

#Import from a backup:
docker run -i -v ~/.kube:/home/argocd/.kube --rm argoproj/argocd:$VERSION argocd-util import - < backup.yaml
```
