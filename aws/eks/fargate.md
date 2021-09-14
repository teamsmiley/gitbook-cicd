# fargate

pod를 ec2에서 실행하지 않고 fargate 에서 실행하고 싶으면 fargate profile을 만들어주면서 조건을 준다.

그 조건에 맞는 pod는 fargate에서 만들어준다.

```text
1vm -> 1 pod
```

vm과 pod가 아이피가 같다.

fargate가 항상 싼건 아니니 테스트를 해서 결정을 해야할듯

## fargate 용 cluster만들기

```text
export AWS_PROFILE=rendercore

echo $AWS_PROFILE

eksctl create cluster \
--name fargate01 \
--version 1.20 \
--region us-west-1 \
--kubeconfig ~/.kube/aws-fargate01 \
--fargate
```

Fargate profile 만들기

```text
eksctl create fargateprofile \
--name rendercore-prod \
--namespace rendercore-prod \
--region us-west-1 \
--cluster cluster01
```

또는 아래처럼 설정으로 만든다. 설정을 사용하면 여러개의 셀렉터를 선택할수 있다.

{% code title="fargateProfile.yml" %}

```text
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: fargate01
  region: us-west-1

fargateProfiles:
  - name: fp-all-namespace
    selectors:
      - namespace: kube-system
      - namespace: argocd
```

{% endcode %}

selector가 5개 이상이면 에러나더라. namespace로만 하지말고 tag 로 처리를 해야할듯

```sh
eksctl create fargateprofile -f fargateProfile.yml

eksctl delete fargateprofile --name fp-all-namespace --cluster cluster01
```

## tip

kube-system을 fargate 로 넣으면 잘 안된다.

`2021-08-03 10:42:33 [ℹ] "coredns" is now scheduled onto Fargate`

이거만 계속 나온다.

원래 있던 노드에서 pod를 지워버리면 fargate로 넘어간다.

```text
Now we have to delete and re-create any existing pods so that they are scheduled on Fargate nodes. Otherwise, pods including the "coredns" pods, will be stuck in "Pending" state.
```
