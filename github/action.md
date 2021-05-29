# GitHub Action

github action이 뭐냐...코드를 커밋/푸시 하면 그 후에 뭔가를 하고 싶다. ci/cd 등등 꼭 ci/cd가 아니더라고 컴파일후 컴파일된 파일을 받아보고 싶거나 s3에 올리고 싶다 등의 무슨일이든 하고 싶은 걸 자동화 해주는 툴이다.

gitlab ci/cd도 같은 기능을 한다. 저는 gitlab ci/cd가 너무 좋아서 그것만 썼는데 이번에 github를 써야할일이 생겨서 확인해봤더니 gitlab의 기능을 대부분 가져서 구현해 둿다.

역시 이바닥은 좋은기능은 다 베끼는..^^ 참고로 gitlab ci/cd가 먼저 나왔다.

해보자.

## workflows

프로젝트에 `.github/workflows` 라는 폴더를 만들고 거기에 build.yml을 만들어 보자.

{% tabs %}
{% tab title="First Tab" %}

{% endtab %}

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

build.yml

```text
name: CI

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: ls -alF
        run: |
          ls -alF
          pwd
```

이제 커밋 푸시해보자.

github웹사이트에 action페이지에 가보자.

![](../.gitbook/assets/github-action-01.png)

안보이던게 생기고 성공햇다고 보여준다.

확인해보자. create build.yml을 클릭

![](../.gitbook/assets/github-action-02.png)

화면을 보면 `ls -alF`를 했고 `pwd`를 해서 현재 폴더를 프린트햇다.

커밋시 뭔가를 해보는거까지는 성공

yml을 설명을 좀 해보면

```text
on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]
```

push나 Pull\_request에 main 브랜치나 dev브랜치에만 이 workflow가 동작한다.

이미지는 ubuntu-latest 를 가지고 빌드를 시작한다.

`name`은 사람들이 보기 편하게 써주면 될거같고 `run`으로 실제 커맨들를 실행한다.

대충 이해햇으니 다음단계로 가보자.

## sample project 생성

프로젝트에 angular 를 하나 추가해보자.

```bash
cd Desktop/GitHub
ng new github-action
cd github-action
nb build --prod
```

커밋/push

## 소스코드를 checkout

```text
- name: Checkout
  uses: actions/checkout@v2
```

추가하고 커밋/푸시 해보자.

![](../.gitbook/assets/github-action-03.png)

runner가 `git checkout` 을 하고 ls -alF를 해서 내용이 확인된다.

![](../.gitbook/assets/github-action-04.png)

## build

1. nodejs 12 버전을 이용
2. @angular/cli 가 글로벌로 설치
3. npm package install
4. project build

```text
- name: Checkout
  uses: actions/checkout@v2

- name: Node 12.x
  uses: actions/setup-node@v2.1.5
  with:
    node-version: '12.x'

- name: install @angular/cli && npm install && ng build
  run: |
    sudo npm install -g @angular/cli
    npm install
    ng build --prod
```

커밋/푸시 하고 액션을 체크해보자.

설명을 하면 `actions/setup-node@v2.1.5` 이것만 설명하면될듯 싶다.

github action은 마켓 플레이스에 사람들이 만들어서 특정 액션을 공유해둔곳이 있다. 아주 많은 부분들이 벌써 만들어져 있어서 그걸 가져다 사용할수 있어서 아주 편하다. 개인들이 만든것도 있고 특별한것들은 깃허브에서 직접 만들어둔게 있다.

[https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)

위 커맨드는 nodejs환경을 구축해주는 액션인데 가져다 쓰면 된다.

![](../.gitbook/assets/github-action-05.png)

![](../.gitbook/assets/github-action-06.png)

마지막 @ 다음에는 버전을 쓰면되는데 최신 내용을 보고싶으면 깃헙 페이지를 확인해보면 된다.

![](../.gitbook/assets/github-action-07.png)

버전을 골라서 사용하면된다.

빌드가 완료됬으니 확인해보자.

![](../.gitbook/assets/github-action-08.png)

node 12.x가 잘 설치가 되었으며 커맨드들도 잘 실행이 된것을 볼수 있다.

![](../.gitbook/assets/github-action-09.png)

## artifact \(결과물\)을 받아보자

빌드하고나면 dist폴더에 결과물이 생긴다 이걸 액션 페이지에서 다운받을수 있게 해보자.

```text
- name: install @angular/cli && npm install && ng build && cd dist
  run: |
    sudo npm install -g @angular/cli
    npm install
    ng build --prod
    pwd
    ls

- uses: actions/upload-artifact@v2
  with:
    name: github-action
    path: dist/github-action/
```

현재 위치를 알기 위해 pwd 와 ls를 실행해봤다.

해보자.

![](../.gitbook/assets/github-action-10.png)

현재 폴더를 `/home/runner/work/github-action/github-action` 이고 dist가 빌드되서 생성된것을 확인할수 있다.

![](../.gitbook/assets/github-action-11.png)

Artifact가 업로드 된것을 알수 있다.

클릭해서 다운로드 받아서 내용을 확인해보자.

![](../.gitbook/assets/github-action-12.png)

정확히 빌드된것을 알수 있다.

대충 이정도로 쓰면 됩니다. 추가로 뭐를 해볼가?

빌드후 s3에 업로드를 해볼가요?

## main브랜치 말고 dev브랜치

일단 프로젝트에 dev 브랜치를 만든다.

![](../.gitbook/assets/github-action-13.png)

```text
on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]
```

dev시에 트리거가 될수있게 추가해준다.

참고로 특정 액션이 특정 브랜치에만 실행이 되야하면 if를 사용할수 있다. `if: github.ref == 'refs/heads/dev'`

```text
- name: replace staging image version number to sha
  if: github.ref == 'refs/heads/dev'
  run: |
    cd apps/w2ps-staging
```

## s3 업로드

일단 마켓 플레이스에서 s3관련 플러그인을 찾는다.

[https://github.com/marketplace/actions/s3-sync](https://github.com/marketplace/actions/s3-sync)

이게 좋을거같다.

![](../.gitbook/assets/github-action-14.png)

해보자.

s3에 버킷을 하나 만들어두고 유저를 생성해서 s3 full권한을 준다.

```text
- uses: jakejarvis/s3-sync-action@v0.5.1
  with:
    args: --acl public-read --follow-symlinks --delete
  env:
    AWS_S3_BUCKET: BUCKET_NAME
    AWS_ACCESS_KEY_ID: YOUR_KEY
    AWS_SECRET_ACCESS_KEY: YOUR_ACCESS
    AWS_REGION: 'us-west-1' # optional: defaults to us-east-1
    SOURCE_DIR: 'dist/github-action/' # optional: defaults to entire repository
```

이정도 하면 s3로 업로드가 된다.

## argocd처럼 다른 github 프로젝트에 커밋하기

빌드하고 artifact를 업로드 하고 argocd에 프로젝트에 커밋을 해야하는경우 새프로젝트를 다시 체크아웃 받고 필요한 작업을 하고 난후 다시 커밋하면된다.

여기에서 PAT라는게 잇는데 이건 github person access token이라는것이다. user &gt;&gt; setting에 가면

developer setting이 잇는데 그걸 누르면 access token을 만들수 있다. 만들때 권한을 줘야하는데 repo정도면 충분해보인다. 생성후 $ 네 넣어주거나 setting에 넣어줘도 된다.

```text
- name: Checkout argocd
  uses: actions/checkout@v2
  with:
    repository: wnwusa/argocd
    token: ${{ secrets.PAT }}

- name: replace staging image version number to sha
  if: github.ref == 'refs/heads/dev'
  run: |
    cd apps/staging
    sed "s/:latest/:${{ github.sha }}/g" 21.deployment-php.origin > 21.deployment-php.yml

- name: replace production image version number to sha
  if: github.ref == 'refs/heads/master'
  run: |
    cd apps/prod
    sed "s/:latest/:${{ github.sha }}/g" 21.deployment-php.origin > 21.deployment-php.yml

- name: Commit files
  run: |
    git config --local user.email "action@github.com"
    git config --local user.name "GitHub Action"
    git commit -am "change docker tag"
    git push
```

