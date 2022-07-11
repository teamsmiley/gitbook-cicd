# tls renew

## 수동

```sh
ssh gitlab.AAA.com
cd /data/git/docker/gitlab
docker exec -it gitlab bash
gitlab-ctl reconfigure
# or
# gitlab-ctl renew-le-certs

```
