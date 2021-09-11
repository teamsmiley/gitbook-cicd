# subchart

## helm subchart

helm 기본 사용법은 다른곳에서 확인해보기 바란다.

그런데 gitops를 지향하는 나로써는 helm쓰기가 참 싫었다.

git으로 커밋을 하지않기 때문에 변경사항을 찾기가 복잡하다. 그러다 오늘 sub chart라는 기능을 확인했다.

차트를 내가 만드는것이다. 이 차트는 깃으로 관리가 될것이고 필요한 어플리케이션은 dependency로 설치하는 것이다.

argocd repo에 curity라는 폴더를 추가한다.

다음에 두개의 파일을 추가한다.

* chart.yaml
* values.yaml

## chart.yaml

파일을 작성해보자.

```yaml
apiVersion: v2
name: curity-subchart
type: application
version: 1.0.0
appVersion: '1.0.0'
dependencies:
  - name: idsvr
    version: 0.9.26
    repository: https://curityio.github.io/idsvr-helm/
```

dependency부분을 제외하고는 마음대로 만드셔도 된다.

dependencies에 내가 사용하고자 하는 helm에 관한 정보를 넣는다.

## values.yaml

모든 helm에는 values.yaml이 있다. 그걸 복사해서 가져온다. 그런데 우리는 depency로 사용하는것이므로 맨 윗줄에 dependency name을 넣어야한다. 그후 아래 설정은 2칸을 밀어준다.

원본

```yaml
replicaCount: 3

image:
  repository: curity.azurecr.io/curity/idsvr
  tag: 6.4.1
  pullPolicy: IfNotPresent
  pullSecret:
```

수정후

```yaml
idsvr:
  replicaCount: 3

  image:
    repository: curity.azurecr.io/curity/idsvr
    tag: 6.4.1
    pullPolicy: IfNotPresent
    pullSecret:
```

이제 커밋하고 argocd에서 git repo로 등록하면 된다.

이제부터는 git 커밋을 통해서 helm옵션들이 업데이트되므로 gitops 철학에 맞다.

