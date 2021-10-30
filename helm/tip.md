# tip

## helm

```sh
helm repo add hazelcast https://hazelcast-charts.s3.amazonaws.com/

helm repo update

helm repo list

helm search repo hazel # hazel이 포함된 모든것이 나온다.

helm list -a
```

## helm template

helm이 적용된후 나오는 yaml을 받아볼수 있다.

```bash
helm template <name> <chart> -f values.yaml --output-dir ~/Desktop/template/

helm template dc-idsvr curity/idsvr -f values.yaml --output-dir ~/Desktop/template/
```

it creates all yml into output directory

## helm values

helm values값을 가져올수 있다.

```bash
helm show values loki/loki-stack > ~/Desktop/values.yaml
helm show values loki/loki-stack --version 2.4.1 > ~/Desktop/values.yaml # version
```
