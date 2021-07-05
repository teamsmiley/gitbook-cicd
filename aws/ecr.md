# ECR

## ECR

## ecr

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

![](../../.gitbook/assets/2021-04-27-09-17-25.png)

create

![](../../.gitbook/assets/2021-04-27-09-17-48.png)

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
[rendercore-ecr]
aws_access_key_id = AKIA2RRDKDNJV4DLSAFM
aws_secret_access_key = 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
```

#### ecr에 로그인

```bash
aws ecr get-login-password --region us-west-2 --profile perseption-ecr
```

```text
eyJwYXlsb2FkIjoiK2x6MzA2SUtJaGtVRnQ2MFNMamVkeS9QM0h0dUYwL3kzZnFKM3ZsN3pybnNienB0UWtQUHo0MWFaV29TaW56elcvbmFHQVJLdENicWlQcGsvdEpPSkRvemtLMGtua2V5SWJweXUyWk5UNFlXYTNxa25Ua1ZmeHpwdTRkZkNDSlFRWWxrV2dCbXkvRFlqSmRQRFo3VGR4YzhPZVQ5OFg1eXNHVFlBMDhNVU0xVzArSkFmd1JDWW5UbUV0ZS9jQjUyVUJJb3MxNmt4YnpRQzkxa0N3TFhHakFYcWIrOXBKSG9CaURhM1pGRmFuU1RjNTF1VlV4MnU5YkxpNmNKZ0laM1ErQmdqamhhRzcwaC9ncm9qQS91SG1oeTJvcExLcUZIODlrckJtMkZmWUM1a2VMeWVjaTlxR0UvVVBEdjNxRmhmc2lCUUwydnNTenhNcHU5cC9sT2tMZTI3ZlZwd2F6aXMzdW9sQWRTRzE3blVZNUhLMFhVSy9ESmpEM0N4SEt2OW8vNVJpcWgzZ2I2blM2cWljY0hzbXZEUlpQWFBoTnU1cVBVelFXcGg0OWN2Qk5Xd2tzRi9mcGF3RklweUJsYjQvajlHVnhZcnMxRkZNUXYrWUpjMnlrQTNqYUQ1WEpzV3NocHB0cm5lOTRQZElKQUxJRU9tQTZNWFZTRGVUMk12VUVGQXZkZGIvTWxIR3JreXJWd2hZV2RSMkxUbTJnUlZIaEp0M1JvV3dZeFU2Sy9EbzJlemw3c0ErSVA2LzVNR2NCTnVOcFl0RDR0dVFiTXZiMEZ2bGgxb2VIMHZUVWkvdERMb0ZxL3RhYVQxU1prcFdHNWg1RUsvY1poY3Z1N29xSmQ1aklRa0dMNkg3MWpjZ1RNdG10bFlwNTNDUG9sbVA0K0x4cUhIbkdGTzV3ZUlXQ3VjK0dpMGRCbi9YbVNXc3ZWbUZpUXRIT0RGQmhzU0MzNmJSV0F0bHNnYjN5cU5nZFQrMFM3aUxzcFpKK2kxVzJQZmJBVGhXbXJXczVmblNLZ3FRakltSTdDSUt6NU4zM2ViTUZ5cjgzS21KV29qZDR2NGlla1BzenVBMHc4YTEzMEFwcDN0K2ZYUk9VTjF4L0NuQUQ1YTJoeCt4SzBkZmVTN2p6SmV1QjFjTkt3ZWVxNHliMEg4dGx0d2FycVBzT2IrTDJVNEpQaUI1NUxGaVQ5dzRjcUtTWW5WK0UvRXRxblBBSTNyZkJqeXZiQjhiVC9wMEo5WDNsTDFIN1VweWVPSk1EUWxZVDFCL08yR0l4LzdjUG9PT3ZtUTl3dEJzcFRqL3dwN3JVdmkrR29zYXcydjFFK1p4R3lYbFB1TGh2Y0ZoVXZLdjZLUkR0V3ZMR09menpVejVRdlRiUTliM2hYVSthVnFReGJZT0tTQjRBVjdod2sxK1N1Q04yM1VKOGNuOTFYdTFLUkhMeE8ra0txWVk0TndIRThGeTQyZjNiN01OQWF6WENnIiwiZGF0YWtleSI6IkFRRUJBSGo2bGM0WElKdy83bG4wSGMwMERNZWs2R0V4SENiWTRSSXBUTUNJNThJblV3QUFBSDR3ZkFZSktvWklodmNOQVFjR29HOHdiUUlCQURCb0Jna3Foa2lHOXcwQkJ3RXdIZ1lKWUlaSUFXVURCQUV1TUJFRURNaHd5Zlo2ZzkrTjV3TGx1UUlCRUlBN0Nka2hMejVMMnNwbHRhYmV3Lzg3eEtwdUhMbGdjU3d2aE9hTU1zYktPM3BabnFaVW9kdExIREkrbjQ0UHdpNjJBMVFRU3dKSEdzcnduTWc9IiwidmVyc2lvbiI6IjIiLCJ0eXBlIjoiREFUQV9LRVkiLCJleHBpcmF0aW9uIjoxNjE5NTg0MzY5fQ==
```

이 비번을 사용해서 도커 로그인을 하면된다.

편하게 하기 위해서는 registry=724849793875.dkr.ecr.us-west-2.amazonaws.com

```bash
aws ecr get-login-password \
    --region us-west-2 \
    --profile stanleyprep-ecr \
| docker login \
    --username AWS \
    --password-stdin ${registry}
```

user는 AWS 이다. 대문자.

![](../../.gitbook/assets/2021-04-27-09-38-19.png)

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
