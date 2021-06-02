# eks 관리

### delete eks

```bash
eksctl delete cluster --name cluster01 --wait
```

### check cluster

```bash
eksctl get cluster
```

### 기타 사용법

```bash
eksctl get nodegroup --cluster=ekc-cluster-1
# 노드 확장
eksctl scale nodegroup --cluster=ekc-cluster-1  --nodes=2 --name=c1-nodes
eksctl scale nodegroup --cluster=ekc-cluster-1  --nodes=3 --nodes-max=3 --name=c1-nodes
eksctl scale nodegroup --cluster=<clusterName> --nodes=<desiredCount> --name=<nodegroupName> [ --nodes-min=<minSize> ] [ --nodes-max=<maxSize> ]
```

### manage cluster

```text
eksctl scale nodegroup --cluster=<clusterName> --nodes=<desiredCount> --name=<nodegroupName> [ --nodes-min=<minSize> ] [ --nodes-max=<maxSize> ]
```

