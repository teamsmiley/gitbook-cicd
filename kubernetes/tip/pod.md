# pod

## delete completed pod

```bash
kubectl delete pod --field-selector=status.phase==Succeeded
```
