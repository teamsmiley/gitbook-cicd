# curity identity server

identity server중에 하나로 새로 배우게 되었다.

## 회원가입후 라이센스 받기

https://developer.curity.io/

커뮤니티 에디션으로 라이센스를 받아서 다운로드해두자.

## arogocd / helm으로 설치

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: curity
  namespace: argocd
spec:
  destination:
    name: ''
    namespace: curity
    server: 'https://kubernetes.default.svc'
  source:
    path: ''
    repoURL: 'https://curityio.github.io/idsvr-helm/'
    targetRevision: 0.9.26
    chart: idsvr
    helm:
      parameters:
        - name: curity.adminUiHttp
          value: 'true'
        - name: curity.config.uiEnabled
          value: 'true'
        - name: curity.config.password
          value: your-password
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

git@github.com:teamsmiley/idsvr-helm.git


## jdbc

curity는 jdbc드라이버를 포함하고 있지 않다. 오라클 라이센스때문에 직접 다운받아서 컨테이너에 넣어줘야한다. 매번 귀찮으니 dockerfile로 만들어서 커스텀 이미지를 사용하자.
