# GTID

## Global Transaction Identifier

각각의 트렌젝션들은 고유한 전역식별자를 갖게 된다.

GTID = source_id:transaction_Id

transaction_id는 해당 서버에서 커밋된 트랜잭션의 순서에 따라 순차적인 숫자로 결정된다.
예로 첫 번째 트랜잭션은 transaction_id=1이 되고, 동일한 서버에서 열 번째 트랜잭션은 transaction_id=10이 된다.
(GTID에서 트랜잭션이 순차적인 숫자는 1부터 시작된다. 0은 될 수 없다.)

## pxc에서 gtid를 백업하기

{% code title="cr.yaml" %}

```yaml
backup:
  pitr:
    enabled: true
    storageName: s3-us-west-binlog
    timeBetweenUploads: 60
  storages:
    s3-us-west-fullbackup:
      type: s3
      s3:
        bucket: xgrid-pxc-cluster01-fullbackup
        credentialsSecret: backup-aws-s3
        region: us-west-1
    s3-us-west-binlog:
      type: s3
      s3:
        bucket: xgrid-pxc-cluster01-binlog
        credentialsSecret: backup-aws-s3
        region: us-west-1
```

{% endcode %}

full backup과 같이 pitr를 활성화 하고 적용하자.

s3에 확인을 해보면 파일들이 백업이 되고 잇는것을 알수 있다.

![](./images/2021-08-23-11-16-29.png)
