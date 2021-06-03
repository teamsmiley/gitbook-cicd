# Repository

## helm 등록

helm chart를 등록하여 앱에 추가하여 사용할수 있다.

argo ui에서 helm repo 등록 : [https://charts.jetstack.io](https://charts.jetstack.io)

![](../.gitbook/assets/argocd-repository-01.png)

![](../.gitbook/assets/argocd-repository-02.png)

![](../.gitbook/assets/argocd-repository-03.png)

![](../.gitbook/assets/image.png)

installCRDs를 false에서 true로 변경하자.

앱 생성하자.

## git repo 등록

git repo를 등록하여 앱에 추가할수 있다.

### argo용 ssh key 생성

```bash
ssh-keygen
> .ssh/argocd
```

![](https://github.com/teamsmiley/modern-ci-cd/tree/8ac743513c1fa98e75444a8dbe175ddb17742576/.gitbook/assets/argocd-repo-04.png)

### repo 등록

* UI

  private키 argocd에 repo에 등록한다. public key는 깃허브에 추가해야할듯 

* cat .ssh/id\_rsaXXXX

![](../.gitbook/assets/argocd-repo-05.png)

![](../.gitbook/assets/argocd-repo-06.png)

만들어진 키의 내용을 복사해다 붙여넣는다.

* Command

```bash
argocd repo add git@github.com:YOURS/argocd.git \
--ssh-private-key-path ~/.ssh/argocd
```

