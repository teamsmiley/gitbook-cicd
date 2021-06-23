# Backup / DR

[https://argoproj.github.io/argo-cd/operator-manual/disaster_recovery/](https://argoproj.github.io/argo-cd/operator-manual/disaster_recovery/)

## EKS용 argocd 도커 작성

매뉴얼대로 하면 aws인증때문에 실패한다. 도커를 가지고 와서 추가 프로그램을 설치 해줘야한다.

일단 도커를 만들어보자.

가급적이면 본인이 사용하는 버전과 같은 버전을 사용한다.

{% code title="Dockerfile" %}

```docker
FROM argoproj/argocd:v2.0.3

USER root

RUN apt-get update
RUN apt install sudo -y
RUN apt install curl -y
RUN apt install unzip -y

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN ./aws/install

RUN curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/aws-iam-authenticator
RUN chmod +x aws-iam-authenticator
RUN mv aws-iam-authenticator /usr/local/sbin/

USER argocd

ENV AWS_PROFILE=xxxx #사용하는 프로파일명
```

{% endcode %}

이제 도커를 빌드해보자.

```bash
cd eks-argocd
docker build -t eks-argocd .
```

이제 여기에 kube 설정과 aws설정을 마운트해주고 export 커맨들르 실행하면된다.

## backup

```bash
docker run -v ~/.kube:/home/argocd/.kube \
-v ~/.aws:/home/argocd/.aws \
--rm \
eks-argocd argocd-util export \
--kubeconfig /home/argocd/.kube/aws-rendercore \
--namespace argocd > backup.yaml
```

## restore

```bash
docker run -v ~/.kube:/home/argocd/.kube \
-v ~/.aws:/home/argocd/.aws \
--rm \
eks-argocd argocd-util import \
--kubeconfig /home/argocd/.kube/aws-rendercore \
--namespace argocd - < backup.yaml
```
