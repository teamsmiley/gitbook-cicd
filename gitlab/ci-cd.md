# gitlab ci/cd

프로젝트 최상위에 .gitlab-ci.yaml 파일을 생성하고, 아래와 같이 설정한다.

## stages 생성

사용할 스테이지를 전부 만들어준다.

```yaml
stages:
  - build
  - testing
  - dev
  - release
  - qa
  - prod
```

## 파이프라인에서 사용할 변수 선언

```yaml
variables:
  IMAGE_NAME: ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}
  LATEST_IMAGE_NAME: ${CI_REGISTRY_IMAGE}:latest
```

## job 생성

```yaml
run_job:
  stage: dev
  image: alpine:3.8
  before_script:
    - echo pre
  script:
    - echo hello world
  after_script:
    - echo post
unit_test_job:
  stage: dev
  image: alpine:3.8
  script:
    - echo unit test
```

run_job을 테스트로 생성햇고 unit_test_job도 생성해보았다.

둘다 stage가 dev인것을 확인할수 있다.

```yaml
run_e2e_tests:
  stage: testing
  image: alpine:3.8
  before_script:
    - echo pre
  script:
    - echo hello world
  after_script:
    - echo post
```

testing stage 에 job을 하나 추가햇다.

## docker build

```sh
echo "hello world" > index.html
vi Dockerfile
```

```dockerfile
FROM nginx:1.20-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["/bin/sh", "-c", "exec nginx -g 'daemon off;';"]
WORKDIR /usr/share/nginx/html
```

.gitlab-ci.yaml 에 추가하자.

build stage 에 job을 하나 추가햇다.

```yaml
build:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  before_script:
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD} ${CI_REGISTRY}
  script:
    - docker build -f Dockerfile -t ${IMAGE_NAME} -t ${LATEST_IMAGE_NAME} .
    - docker push ${IMAGE_NAME}
    - docker push ${LATEST_IMAGE_NAME}
  after_script:
    - docker logout
```

`${xxx}`는 variable에 있는 변수를 사용한다.

CI_REGISTRY_USER, CI_REGISTRY_PASSWORD, CI_REGISTRY 를 사용한다.

![](./images/2022-07-02-17-54-30.png)

그림처럼 생성해서 넣어줘야한다.

혹시 여러사람이 공유하면 그룹 단위로 적용하는게 키를 보호하는데 도움이 될듯 싶다.

## change log

- $CI_COMMIT_TAG => 태그를 붙이는 순간만 동작을 한다.

```yaml
changelog:
  stage: release
  image: docker:git
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - git log $(git describe --tags --abbrev=0 HEAD^)..HEAD --pretty=format:'At %ci, %cN committed %h - %s' --decorate --graph >release.md
  artifacts:
    paths: [release.md]
```

git log를 사용하여 바로 직전 태그 이후로부터 지금까지 git log를 가져오고, release.md로 저장한다.

release.md가 artifactory로 생성이 되서 ui에서 다운받을수 있다.

![](./images/2022-07-02-17-58-30.png)

## release

- $CI_COMMIT_TAG => 태그를 붙이는 순간만 동작을 한다.

바로 직전에 생긴 artifact가 자동으로 다운이 된다. 그러므로 `cat`해서 릴리즈에 넣으면 된다.

needs를 이용하면 디펜던시가 걸린 작업이 먼저 끝난후 이 잡을 실행한다. changelog가 끝나면 release가 된다.

```yaml
release:
  stage: release
  needs: ['changelog']
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - echo "running release_job for $TAG"
  release:
    tag_name: '$CI_COMMIT_TAG'
    name: 'Release $CI_COMMIT_TAG'
    description: '$(cat release.md)'
```

이제 태그를 붙이고 push를 하면 릴리즈 페이지가 된다.

![](./images/2022-07-02-18-01-52.png)

## staging 배포

stage를 staging으로 구성햇음.

argocd repo를 업데이트 하는것으로 배포가 마무리 된다. 깃을 클론해서 다운로드한후 docker image tag만 바꿔주고 다시 커밋하면된다.

```yml
staging-deploy:
  stage: staging
  image: alpine:3.8
  rules:
    - if: $CI_COMMIT_TAG
  before_script:
    - apk add --no-cache git curl bash
    - git config --global user.email "gitlab@gitlab.com"
    - git config --global user.name "GitLab CI/CD"
  script:
    - git clone https://${CI_USER}:${CI_PUSH_TOKEN}@gitlab.com/teamsmiley/staging.git
    - cd k8s-c1/apps/default/sample-www-internal
    - sed "s/:latest/:${CI_COMMIT_SHORT_SHA}/g" deploy.origin > deploy.yaml
    - git commit -am "change docker tag"
    - git push
```

deploy.origin 파일

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: www
  labels:
    app: www
spec:
  replicas: 1
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      app: www
  template:
    metadata:
      labels:
        app: www
    spec:
      containers:
        - name: www
          image: registry.gitlab.com/teamsmiley/sample:latest
          ports:
            - containerPort: 80
              name: http
      imagePullSecrets:
        - name: gitlab-regcred
```

## production 배포(job template)

스테이징 코드를 복사해서 만들면 된다. 그런데 코드의 중복이 생긴다.

여기서 job template라는 개념이 생긴다.

job template는 사용한곳보다는 yaml파일에 상단에 위치해야한다. yaml에 추가하자.

jobname이 .으로 시작한다. 이건 실제로 동작하지는 않는 작업이다.

```yaml
.deploy-template: &template
  rules:
    - if: $CI_COMMIT_TAG
  before_script:
    - apk add --no-cache git curl bash
    - echo ${CI_COMMIT_SHORT_SHA}
    - git config --global user.email "gitlab@gitlab.com"
    - git config --global user.name "GitLab CI/CD"
  script:
    - git clone https://${CI_USER}:${CI_PUSH_TOKEN}@gitlab.com/oomaforbin/oomacorp/networkoperations/staging.git
    - cd staging/apps/default/sample-www-internal
    - sed "s/:latest/:${CI_COMMIT_SHORT_SHA}/g" deploy.origin > deploy.yaml
    - git commit -am "change docker tag"
    - git push
```

yaml에 anchor기능을 이용한다.

`&template` 처럼 `&`하고 이름을 적는다.

이제 사용할 곳에서 이름을 적어서 사용한다. `<<: *` 다음에 이름을 써서 사용한다.

```yaml
staging-deploy:
  stage: staging
  image: alpine:3.8
  rules:
    - if: $CI_COMMIT_TAG
  <<: *template
```

이렇게 하면 위 내용을 가져다가 붙여준다. 코드 중복을 줄일수 있다.

production과 staging을 하다보니 git repo의 경로가 다르다.

staging.git / production.git 이다. 어떻게 처리해야할까?

```yaml
- git clone https://${CI_USER}:${CI_PUSH_TOKEN}@gitlab.com/oomaforbin/oomacorp/networkoperations/staging.git
- cd staging/apps/default/sample-www-internal
```

이부분이 문제인데 이건 간단하게 variable을 사용하면된다.

```yaml
.deploy-template: &template
  rules:
    - if: $CI_COMMIT_TAG
  before_script:
    - apk add --no-cache git curl bash
    - echo ${CI_COMMIT_SHORT_SHA}
    - git config --global user.email "gitlab@gitlab.com"
    - git config --global user.name "GitLab CI/CD"
  script:
    - git clone https://${CI_USER}:${CI_PUSH_TOKEN}@gitlab.com/oomaforbin/oomacorp/networkoperations/$gitrepo_name.git # $gitrepo_name은 variable을 사용하여 정해준다.
    - cd $gitrepo_name/apps/default/sample-www-internal # $gitrepo_name은 variable을 사용하여 정해준다.
    - sed "s/:latest/:${CI_COMMIT_SHORT_SHA}/g" deploy.origin > deploy.yaml
    - git commit -am "change docker tag"
    - git push
```

```yaml
staging-deploy:
  stage: staging
  image: alpine:3.8
  rules:
    - if: $CI_COMMIT_TAG
  # 이부분을 추가한다.
  variables:
    gitrepo_name: staging
  <<: *template

production-deploy:
  stage: production
  image: alpine:3.8
  rules:
    - if: $CI_COMMIT_TAG
  # 이부분을 추가한다.
  variables:
    gitrepo_name: production
  <<: *template
```

이렇게 하면 variable을 사용하여 staging은 staging.git / production은 production.git 이렇게 정해준다.

코드의 중복을 전부 job template를 이용하여 처리할수 있다.

## Approval

environment에서 approval을 사용하면 된다.

```yaml
production-deploy:
  stage: production
  image: alpine:3.8
  # 이부분을 추가한다.
  environment:
    name: prod
    url: https://www.aaa.com
  rules:
    - if: $CI_COMMIT_TAG
  variables:
    gitrepo_name: production
  <<: *template
```

ci/cd를 실행하면 environment가 생성된다.

![](./images/2022-07-02-18-20-55.png)

setting >> ci/cd >> protect an environment

![](./images/2022-07-02-18-51-30.png)

![](./images/2022-07-02-18-52-16.png)

![](./images/2022-07-02-18-52-34.png)

![](./images/2022-07-02-18-53-08.png)

이제 cicd를 실행해보면 프로덕션에서 멈춰있는것을 알수 있다.

![](./images/2022-07-02-18-53-38.png)

![](./images/2022-07-02-18-53-55.png)

environment 에 가서 productin을 선택하고 들어가면 컨펌을 할수가 있다.

![](./images/2022-07-02-18-55-07.png)

![](./images/2022-07-02-18-22-08.png)

![](./images/2022-07-02-18-55-32.png)

approval을 할수가 있다.

approval을 하면 다음 프로세스가 진행되는것을 알수 있다.

## 전체 ci/cd 프로세스

![](./images/2022-07-02-18-57-57.png)

```yaml
stages:
  - build
  - testing
  - dev
  - release
  - qa
  - prod

variables:
  IMAGE_NAME: ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}
  LATEST_IMAGE_NAME: ${CI_REGISTRY_IMAGE}:latest

run_job:
  stage: dev
  image: alpine:3.8
  before_script:
    - echo pre
  script:
    - echo hello world
  after_script:
    - echo post
unit_test_job:
  stage: dev
  image: alpine:3.8
  script:
    - echo unit test

run_e2e_tests:
  stage: testing
  image: alpine:3.8
  before_script:
    - echo pre
  script:
    - echo hello world
  after_script:
    - echo post

build:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  before_script:
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD} ${CI_REGISTRY}
  script:
    - docker build -f Dockerfile -t ${IMAGE_NAME} -t ${LATEST_IMAGE_NAME} .
    - docker push ${IMAGE_NAME}
    - docker push ${LATEST_IMAGE_NAME}
  after_script:
    - docker logout

changelog:
  stage: release
  image: docker:git
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - git log $(git describe --tags --abbrev=0 HEAD^)..HEAD --pretty=format:'At %ci, %cN committed %h - %s' --decorate --graph >release.md
  artifacts:
    paths: [release.md]

release:
  stage: release
  needs: ['changelog']
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - echo "running release_job for $TAG"
  release:
    tag_name: '$CI_COMMIT_TAG'
    name: 'Release $CI_COMMIT_TAG'
    description: '$(cat release.md)'

.deploy-template: &template
  image: alpine:3.8
  rules:
    - if: $CI_COMMIT_TAG
  before_script:
    - apk add --no-cache git curl bash
    - echo ${CI_COMMIT_SHORT_SHA}
    - git config --global user.email "gitlab@gitlab.com"
    - git config --global user.name "GitLab CI/CD"
  script:
    - git clone https://${CI_USER}:${CI_PUSH_TOKEN}@gitlab.com/oomaforbin/oomacorp/networkoperations/staging.git
    - cd staging/apps/default/sample-www-internal
    - sed "s/:latest/:${CI_COMMIT_SHORT_SHA}/g" deploy.origin > deploy.yaml
    - git commit -am "change docker tag"
    - git push

staging-deploy:
  stage: staging
  # 이부분을 추가한다.
  variables:
    gitrepo_name: staging
  <<: *template

production-deploy:
  stage: production
  # 이부분을 추가한다.
  variables:
    gitrepo_name: production
  <<: *template
```

## todo

- approval email
- ci/cd list for all project
- jira 연동
