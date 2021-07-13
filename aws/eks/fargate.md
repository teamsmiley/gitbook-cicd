# fargate

```text
export AWS_PROFILE=rendercore

echo $AWS_PROFILE

eksctl create cluster \
--name cluster02 \
--version 1.20 \
--region us-west-1 \
--kubeconfig ~/.kube/aws-cluster02 \
--fargate
```

```text
eksctl create fargateprofile \
--name rendercore-prod \
--namespace rendercore-prod \
--region us-west-1 \
--cluster cluster01
```





