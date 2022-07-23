# Table of contents

- [Modern DEVOPS](README.md)

## AWS

- [AWS CLI](aws/cli.md)
- [EKS](aws/eks/README.md)
  - [cluster manage](aws/eks/manage-cluster.md)
  - [ALB Controller](aws/eks/alb-controller.md)
  - [external-dns](aws/eks/external-dns.md)
  - [fargate](aws/eks/fargate.md)
- [ECR](aws/ecr.md)
- [S3](aws/s3.md)
- [Certificate Manager](aws/certificate-manager.md)

## Argocd

- [Install](argocd/install.md)
- [CLI](argocd/cli.md)
- [Repository](argocd/repository.md)
- [Apps](argocd/apps.md)
- [AWS ALB 사용](argocd/aws-alb.md)
- [notification slack](argocd/notification-slack.md)
- [Backup / DR](argocd/backup-dr.md)
- [ingress](argocd/ingress.md)
- [2021-11-16 Github error](argocd/github-error.md)

## curity identity server

- [curity](curity-identity-server/curity.md)

## Docker

- [Registry](docker/registry.md)

## DNS Bind9

- [DNS Bind9](dns-bind9/README.md)
  - [Dynamic Update](dns-bind9/dynamic-update.md)
  - [Log](dns-bind9/log.md)
  - [Cert-Manager](dns-bind9/cert-manager.md)
  - [Certbot](dns-bind9/certbot.md)

## DB

- [PXC](db/percona-xtra-cluster/README.md)
  - [Operator](db/percona-xtra-cluster/pxc-operator.md)
  - [PMM](db/percona-xtra-cluster/pmm.md)
  - [삭제](db/percona-xtra-cluster/pxc-delete.md)
  - [GTID](db/percona-xtra-cluster/gtid-global-transaction-identifier.md)
  - [Cross Site Replication](db/percona-xtra-cluster/cross-site-replication.md)
- [Mssql](db/mssql.md)
- [mysql](db/mysql.md)

## GIT

- [basic](git/basic.md)
- [submodule](git/submodule.md)

## GitHub

- [Repository](github/repository.md)
- [GitHub Action](github/action.md)
- [GitHub PR](github/pull-request.md)

## GitLab

- [CI/CD](gitlab/ci-cd.md)
- [ssl renew](gitlab/tls-renew.md)

## Helm

- [subchart](helm/subchart.md)
- [tip](helm/tip.md)

## Kubernetes

- [Kubernetes는 무엇인가](kubernetes/kube-basic.md)
- [tools](kubernetes/tools.md)
- [install with kubespray](kubernetes/install-with-kubespray.md)
- [Extension](kubernetes/extension/README.md)
  - [longhorn - pvc](kubernetes/extension/longhorn-pv.md)
  - [external-dns](kubernetes/extension/external-dns.md)
  - [ingress-nginx](kubernetes/extension/ingress-nginx.md)
  - [Cert-Manager](kubernetes/extension/cert-manager.md)
  - [kube-prometheus]kubernetes/extension/kube-prometheus.md)
- [TIP](kubernetes/tip/README.md)
  - [job](kubernetes/tip/job.md)
  - [pod](kubernetes/tip/pod.md)
  - [log](kubernetes/tip/log.md)

## Linux

- [export and variable](linux/export-and-variable.md)
- [grep 사용법](linux/grep.md)

## MAAS

- [install maas](maas/install-maas.md)
- [manage maas](maas/manage-maas.md)
- [tip](maas/tip.md)

## OPENNEBULA

- [install](opennubula/README.md)
- [tip](opennubula/tip.md)
- [install-ansible](opennubula/ansible.md)

## Prometheus
- [TIP](prometheus/tip.md)
- [loki](prometheus/loki.md)
