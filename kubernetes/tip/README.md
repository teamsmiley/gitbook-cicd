# TIP

## delete completed pod

```bash
kubectl delete pod --field-selector=status.phase==Succeeded
```

## delete completed job

```bash
kubectl delete jobs --field-selector status.successful=1
```

## delete fail job or long running job

```bash
kubectl delete jobs --field-selector status.successful=0
```

