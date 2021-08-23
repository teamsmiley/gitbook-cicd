# percona extra db cluster operator

mysql을 설치해주고 복제해주고 모든걸 자동으로 해주는것이 목표

pv/pvc가 없이 tempdir 이나 hostpath로도 테스트는 가능하나 백업/복구 등에 문제가 있다.

현재는 모든 스크립트가 pv/pvc가 있는것을 가정하는것 같다. 그러므로 앞에 설치한 longhorn을 설치를 해두고 작업하기를 바란다

## percona-xtradb-cluster-operator

```bash
git clone -b v1.8.0 https://github.com/percona/percona-xtradb-cluster-operator

cd percona-xtradb-cluster-operator/deploy

k create namespace pxc-mysql

kcn pxc-mysql
k get pod
k apply -f crd.yaml
k apply -f rbac.yaml
k apply -f operator.yaml
# 또는 합쳐져있는 k apply -f bundle.yaml
```

## password setting

이제 관련 비밀번호등을 설정하자. 비번을 전부 바꾼다. 적용한다.

`cat secrets.yaml`

```text
apiVersion: v1
kind: Secret
metadata:
  name: my-cluster-secrets # 이름 중요
type: Opaque
stringData:
  root: your-password
  xtrabackup: your-password
  monitor: your-password
  clustercheck: your-password
  proxyadmin: your-password
  pmmserver: your-password
  operator: your-password
  replication: your-password
```

적용하자.

```bash
k apply -f secrets.yaml
```

aws에 백업을 하기 위한 비번도 필요하다.

`cat backup-s3.yaml`

```text
apiVersion: v1
kind: Secret
metadata:
  name: backup-s3 # 이름
  namespace: pxc-mysql
type: Opaque
data:
  AWS_ACCESS_KEY_ID: QUtJQVxxxjc=
  AWS_SECRET_ACCESS_KEY: VktqdzZWTjRDMjxxxY5MUQ5OQ==
```

```bash
k apply -f backup-s3.yaml
```

## 디비 디플로이

{% code title="cr.yaml" %}

```yml
secretsName: my-cluster-secrets # secret.yml에 있는 이름을 넣어줘야함.
allowUnsafeConfigurations: true # tls 통신안쓰는것으로 처리
haproxy:
  enabled: false # proxysql을 사용할 예정
proxysql:
  enabled: true
  serviceType: LoadBalancer
pmm:
  enabled: true
  image: percona/pmm-client:2.18.0
  serverHost: pxc-pmm-service # pmm-server에서의 서비스 명
  serverUser: admin # 확인
backup: # backup 설정
  storage:
    s3:
      bucket: S3-BACKUP-BUCKET-NAME-HERE #s3 bucket name
      credentialsSecret: backup-s3 # backup-s3.yaml에 있는 이름
      region: us-west-1 # s3 bucket 위치
    #fs-pvc: # 사용하지 않음
    schedule: #스케줄을 적용
      - name: 'sat-night-backup'
        schedule: '0 0 * * 6'
        keep: 52
        storageName: s3-us-west
      - name: 'daily-backup'
        schedule: '0 0 * * *'
        keep: 7
        storageName: s3-us-west
      - name: 'hourly-backup'
        schedule: '0 * * * *'
        keep: 24
        storageName: s3-us-west
```

{% endcode %}

proxysql을 사용하고 haproxy를 사용하지 않음.

백업 설정

적용

```bash
k apply -f cr.yaml
k get svc # loadbalance ip확인
```

proxysql 로드 발란스 아이피로 디비에 접속해보면 된다. \( 172.16.4.157 \)

아니면 다음 커맨드를 사용한다.

```bash
kubectl run -i --tty --rm percona-client --image=percona --restart=Never \
  -- mysql -h cluster02-pxc.pxc-mysql.svc.cluster.local -uroot -p
#type your password
```

cluster02-pxc.pxc-mysql.svc.cluster.local - $service-name.$namespace.svc.cluster.local

디비에 테스트 데이터를 넣어보자.

```sql
create database test;
use test;
create table movies(id int auto_increment primary key, name varchar(20) not null);
show tables;
insert into movies(name) values('hello1');
select * from movies;
```

이제 각각의 pod에서 잘 작동하는지 확인해보자.

`k get pod`에서 아이피를 찾아서 아이피로 접속해본다.

- 10.233.111.105
- 10.233.118.212
- 10.233.108.255

```bash
kubectl run -i --tty --rm percona-client0 --image=percona --restart=Never \
 -- mysql -h 10-233-111-105.pxc-mysql.pod.cluster.local -uroot -p
#type your password

kubectl run -i --tty --rm percona-client1 --image=percona --restart=Never \
 -- mysql -h 10-233-118-212.pxc-mysql.pod.cluster.local -uroot -p
#type your password

kubectl run -i --tty --rm percona-client2 --image=percona --restart=Never \
 -- mysql -h 10-233-108-255.pxc-mysql.pod.cluster.local -uroot -p
#type your password
```

모두 접속하여

```bash
select * from movies
```

를 실행해서 넣은 데이터가 들어가 있는지 확인 한다.

![](images/2021-08-20-17-47-41.png)

잘 복제되고 있는것을 확인하였다.

## 백업

- 자동 백업 백업 스케줄을 해두었음로 한시간에 한번씩 s3 bucket으로 업로드 된다.
- 수동 백업 수동으로 백업을 받고 싶으면 yml을 수정하고 적용하면된다.

```bash
cat > backup.yaml <<EOF
apiVersion: pxc.percona.com/v1
kind: PerconaXtraDBClusterBackup
metadata:
  #  finalizers:
  #    - delete-s3-backup #지울때 s3 backup까지 같이지우고 싶은경우는 사용하자.
  name: backup1 # 이름을 잘 설정하자.
spec:
  pxcCluster: cluster01
  storageName: s3-us-west
EOF
```

```bash
kubectl apply -f backup/backup.yaml
```

s3에 업로드 된것을 확인할수 있다.

[https://www.percona.com/doc/kubernetes-operator-for-pxc/backups.html\#making-on-demand-backup](https://www.percona.com/doc/kubernetes-operator-for-pxc/backups.html#making-on-demand-backup)

## 복구

```bash
cat > restore.yaml <<EOF
apiVersion: pxc.percona.com/v1
kind: PerconaXtraDBClusterRestore
metadata:
  name: restore1
spec:
  pxcCluster: cluster01
  backupName: backup1
EOF

kubectl apply -f backup/restore.yaml
```

클러스터를 하나씩 없애고 복구하고 다시 올려준다.

## DR

일단 노드 1개가 고장나면 자동으로 다른노드에 올려주고 이 노드에서는 자동으로 master에서 파일을 가져와서 복구를 시간한후 자동으로 붙여준다.

만약 3개 노드가 동시에 꺼저버리면 문제가 될듯 보인다. 그래서 찾아봣더니 3개 노드중 마지막 데이터가 잇는곳을 찾아서 그곳을 마스터로 지정하고 난후 나머지 2개 노드를 다시 자동으로 올려준다고하니 큰 문제는 없어 보인다.

![](../.gitbook/assets/2021-08-19-07-03-12.png)

[https://youtu.be/V3ko5NpTMPA?t=895](https://youtu.be/V3ko5NpTMPA?t=895)

longhorn에서 스토리지에 리플리카를 지원을 하므로 3개 정도 해두거나 전체 노드 댓수에 해두면 전체 노드에 같은 데이터가 잇는것이므로 어느 노드에서 실행이 되더라도 자동으로 붙여서 올라올것으로 보인다.

## 결론

총 3대의 mysql 클러스터가 구성이 되었고 longhorn을 사용하여 데이터가 쿠베 클러스터에 저장이 된다. 백업은 s3로 자동으로 업로드가 된다. pmm으로 모니터링이 가능하다.

## 알고 있는 문제

가끔 테스트한것들을 지우려고 할때 행걸리는 경우가 있다. backup이 안지워져서 그런 경우가 있음

pxc-backups로 검색해서 edit 해서 finalize를 지워줘야 한다.

외부 스토리지를 지우지 못해서 행이 걸리는건데 이 부분을 무시하고 지날수 있다.
