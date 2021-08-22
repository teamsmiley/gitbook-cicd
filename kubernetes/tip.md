# TIP

## delete completed pod

```sh
kubectl delete pod --field-selector=status.phase==Succeeded
```

## delete completed job

```sh
kubectl delete jobs --field-selector status.successful=1
```

## delete fail job or long running job

```sh
kubectl delete jobs --field-selector status.successful=0
```
