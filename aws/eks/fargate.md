# fargate

pod를 ec2에서 실행하지 않고 fargate 에서 실행하고 싶으면 fargate profile을 만들어주면서 조건을 준다. 

그 조건에 맞는 pod는 fargate에서 만들어준다.  

1vm -&gt; 1 pod 

vm과 pod가 아이피가 같다. 

fargate가 항상 싼건 아니니 테스트를 해서 결정을 해야할듯  

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

```text
eksctl create fargateprofile \
--name rendercore-prod \
--namespace rendercore-prod \
--region us-west-1 \
--cluster cluster01
```



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
      - namespace: default
      - namespace: kube-system
      - namespace: argocd
```
{% endcode %}

```text
eksctl create fargateprofile -f fargateProfile.yml
```

