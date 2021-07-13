# fargate

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





