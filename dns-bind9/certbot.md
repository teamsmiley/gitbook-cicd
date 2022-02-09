# Certbot

certbot을 사용하여 lets encrypt ssl을 발급해보자. 

## docker

docker를 설치

## create ini file

vi rfc2136.ini

```conf
# Target DNS server (IPv4 or IPv6 address, not a hostname)
dns_rfc2136_server = 172.21.1.20
# Target DNS port
dns_rfc2136_port = 53
# TSIG key name
dns_rfc2136_name = teamsmiley-dev-secret
# TSIG key secret
dns_rfc2136_secret = KzqRA3OnnS3Awp9mlLFWhRziEsZE7qB/YyiZrPl1ow==
# TSIG key algorithm
dns_rfc2136_algorithm = HMAC-SHA256
```

## renew ssl 

