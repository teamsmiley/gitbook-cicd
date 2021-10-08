# pod

## delete completed pod

```bash
kubectl delete pod --field-selector=status.phase==Succeeded
```


## env를 configmap으로 이용하기

configmap 이 있는 상황에서 pod에서 env값으로 configmap을 이용하기

```text
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ['/bin/sh', '-c', 'env']
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
```

## envFrom를 configmap으로 이용하기

```text
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ['/bin/sh', '-c', 'env']
      envFrom:
        - configMapRef:
            name: special-config
  restartPolicy: Never
```

이렇게 하면 configmap에 있던 모든 내용이 env값으로 변환된다.

