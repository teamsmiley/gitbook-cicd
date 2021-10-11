# tip

## helm template

```bash
helm template <name> <chart> -f values.yaml --output-dir ~/Desktop/template/

helm template dc-idsvr curity/idsvr -f values.yaml --output-dir ~/Desktop/template/
```

it creates all yml into output directory

## helm values

```bash
helm show values loki/loki-stack > ~/Desktop/values.yaml
helm show values loki/loki-stack --version 2.4.1 > ~/Desktop/values.yaml # version
```
