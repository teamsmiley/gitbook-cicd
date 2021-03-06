# ECR

## ECR이란?

aws docker registry

- 기본 프라이빗 레지스트리의 URL은 입니다
  - [https://aws_account_id.dkr.ecr.region.amazonaws.com](https://aws_account_id.dkr.ecr.region.amazonaws.com)

### registry vs repository

- aws_account_id.dkr.ecr.region.amazonaws.com/AAA
  - aws_account_id.dkr.ecr.region.amazonaws.com 이것이 registry
  - AAA가 repository

### iam

aws console에서 사용하려다 보면 permission이 막힌다.

iam 에서 ecr-user를 생성 다음 롤을 준다.

- Programmatic access : on
- direct attach role
  - AmazonEC2ContainerRegistryFullAccess

키를 다운받아둔다 나중에 쓴다.

### create repository

click create repository

private

name

![](https://github.com/teamsmiley/gitbook-cicd/tree/6c6af90bd39b78b1cb0fea262784d32af3a17216/.gitbook/assets/2021-04-27-09-17-25.png)

create

![](https://github.com/teamsmiley/gitbook-cicd/tree/6c6af90bd39b78b1cb0fea262784d32af3a17216/.gitbook/assets/2021-04-27-09-17-48.png)

### 이게 도커이미지를 push 하자

랩탑에서

```bash
aws ecr help
```

iam 에서 만든 유저를 등록

```bash
aws configure

AWS Access Key ID [None]: YOURACCESSKEY
AWS Secret Access Key [None]: YOURSECRETKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

```text
vi ~/.aws/credential
[ecr]
aws_access_key_id = AKIAxx2RRDKDNJV4DLSAFM
aws_secret_access_key = EhRxl4qxxjmv71JKVtvhs9SwJeYiPLOLcgRHq7odWZ
```

### ecr에 로그인

```bash
aws ecr get-login-password --region us-west-2 --profile perseption-ecr
```

```text
eyJwYXlsb2FkIjoiK2x6MzA2SUtJaGtVRnQ2MFNMamVkeS9QM0h0dUYwL3kzZnFKM3ZsN3pybnNienB0UWtQUHo0MWFaV29TaW56elcvbmFHQVJLdENicWlQcGsvdEpPSkRvemtLMGtua2V5SWJweXUyWk5UNFlXYTNxa25Ua1ZmeHpwdTRkZkND3eEtwdUhMbGdjU3d2aE9hTU1zYktPM3BabnFaVW9kdExIREkrbjQ0UHdpNjJBMVFRU3dKSEdzcnduTWc9IiwidmVyc2lvbiI6IjIiLCJ0eXBlIjoiREFUQV9LRVkiLCJleHBpcmF0aW9uIjoxNjE5NTg0MzY5fQ==
```

이 비번을 사용해서 도커 로그인을 하면된다.

편하게 하기 위해서는 registry=72484979875.dkr.ecr.us-west-2.amazonaws.com

```bash
aws ecr get-login-password \
    --region us-west-2 \
    --profile aaa-ecr \
| docker login \
    --username AWS \
    --password-stdin ${registry}
```

user는 AWS 이다. 대문자.

![](https://github.com/teamsmiley/gitbook-cicd/tree/6c6af90bd39b78b1cb0fea262784d32af3a17216/.gitbook/assets/2021-04-27-09-38-19.png)

### docker build

```bash
cd ~/Desktop/GitHub/perseption/GitHub/wps-main-web
docker build . -t perseption-www
```

### image upload to ecr

```bash
docker tag perseption-www ${registry}/www:latest
docker push ${registry}/www:latest
```

푸시가 잘 올라가면 된다.

### 이미지들 자동으로 지우기

설정을 해두자.

이미지 갯수로도 된다. 확인해두자.
