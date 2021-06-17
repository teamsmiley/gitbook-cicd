# cluster manage

## delete eks

```bash
eksctl delete cluster --name cluster01 --wait
```

![](../../.gitbook/assets/2021-06-02-05-47-12.png)

![](../../.gitbook/assets/2021-06-02-05-53-17.png)

## cluster 리스트보기

```bash
eksctl get cluster
```

## 기타 사용법

```bash
eksctl get nodegroup --cluster=cluster01

# 노드 확장

eksctl scale nodegroup --cluster=cluster01  --nodes=2 --name=cluster01-nodes

eksctl scale nodegroup --cluster=cluster01  --nodes=3 --nodes-max=3 --name=cluster01-nodes

eksctl scale nodegroup --cluster=<clusterName> --nodes=<desiredCount> --name=<nodegroupName> [ --nodes-min=<minSize> ] [ --nodes-max=<maxSize> ]
```

## manage cluster

```bash
eksctl scale nodegroup --cluster=<clusterName> --nodes=<desiredCount> --name=<nodegroupName> [ --nodes-min=<minSize> ] [ --nodes-max=<maxSize> ]
```

Kubectl 사용

```bash
kubectl get pod --all-namespaces  
kubectl get pod --all-namespaces -o wide
```

전체 pod 갯수 :  

```bash
kubectl get pod --all-namespaces  | wc -l
```

노드당 갯수 

\`kubectl get node\` 를 해서 노드 이름을 확인후 노드별로 체크 

```bash
kubectl get pod --all-namespaces -o wide | grep ip-192-168-10-183 | wc -l
kubectl get pod --all-namespaces -o wide | grep ip-192-168-28-3 | wc -l
kubectl get pod --all-namespaces -o wide | grep ip-192-168-4-220 | wc -l
kubectl get pod --all-namespaces -o wide | grep ip-192-168-65-172 | wc -l
kubectl get pod --all-namespaces -o wide | grep ip-192-168-69-253 | wc -l
kubectl get pod --all-namespaces -o wide | grep ip-192-168-78-242 | wc -l
kubectl get pod --all-namespaces -o wide | grep ip-192-168-9-123 | wc -l
```



