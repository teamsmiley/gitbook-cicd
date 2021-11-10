# vault (csi driver)

## install vault

```sh
brew tap hashicorp/tap
brew install hashicorp/tap/vault

vault #확인

vault server -dev

Unseal Key: U7zswyoUaZJeIKgivGnn5WOzQD+RhAg1jaHcO6e7zZU=
Root Token: s.JroRmH4RS69z1Zto2mmdKgnt

echo "U7zswyoUaZJeIKgivGnn5WOzQD+RhAg1jaHcO6e7zZU=" > unseal.key
export VAULT_DEV_ROOT_TOKEN=s.JroRmH4RS69z1Zto2mmdKgnt

vault status

vault kv get secret/hello
vault kv put secret/hello foo=world

```

## install csi driver

```sh
helm repo add hashicorp https://helm.releases.hashicorp.com
helm search repo hashicorp/vault

#NAME           	CHART VERSION	APP VERSION	DESCRIPTION
#hashicorp/vault	0.17.1       	1.8.4      	Official HashiCorp Vault Chart

helm install vault hashicorp/vault
```

https://www.vaultproject.io/docs/platform/k8s/csi/examples
