# AWS CLI

## aws cli 설치

```sh
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

## AWS 액세스 키 생성

## aws cli setup command

```sh
aws configure
```

생성한 키와 내용을 넣는다.

~/.aws/에 파일이 생성된다

vi ~/.aws/config

```yaml
[smiley]
region = us-west-2
output = yaml
```

혹시 디폴트로 되면 그걸 사용하면 된다.

vi ~/.aws/credentials

```yaml
[smiley]
aws_access_key_id = access-key
aws_secret_access_key = your-secret
```

## 기본 유저 설정

```sh
export AWS_PROFILE=smiley
```

## 확인

```sh
aws help
aws command help
aws command subcommand help
aws ec2 describe-instances help

aws ec2 describe-instances  --region us-west-2 --query "Reservations[].Instances[].InstanceId"
```
