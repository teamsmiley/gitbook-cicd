# Cross Site Replication

두개의 쿠버네티스에서 디비를 리플리케이션 할수 있다.

[https://www.percona.com/doc/kubernetes-operator-for-pxc/replication.html](https://www.percona.com/doc/kubernetes-operator-for-pxc/replication.html)

![](https://github.com/teamsmiley/gitbook-cicd/tree/3ab8992dd8cf9d807f28c18e33148b6458f18afa/db/mysql/images/2021-08-22-23-54-23.png)

## 주의

* Percona XtraDB Cluster 8.0.22+ 이상을 설치해야 한다.
* haproxy를 사용하면 잘되는데 proxy sql을 사용하면 안된다.
* 복제시 사용할 비번이 같아야한다. secret.yml에 같은 비번으로 설정해주면 된다.

## source cluster 구성

`vi cr.yaml`

```yaml
spec:
  pxc:
    expose:
      enabled: true
      type: LoadBalancer
    replicationChannels:
      - name: pxc1_to_pxc2
        isSource: true
```

체널을 설정하고 isSource를 true로 설정한다.

load balance를 이용하여 svc를 통해 외부에 오픈한다.

`kubectl get services -l "app.kubernetes.io/instance=CLUSTER_NAME"` 서비스를 확인하자. 아이피를 확인하자.

## replica cluster

복제를 받을 클러스터에서 설정하자.

위에서 로드발란스로 오픈한 아이피를 사용한다.

`vi cr.yaml`

```yaml
pxc:
  replicationChannels:
    - name: pxc1_to_pxc2
    isSource: false
    sourcesList:
    - host: 172.16.3.155
      port: 3306
      weight: 40
    - host: 172.16.3.156
      port: 3306
      weight: 30
    - host: 172.16.3.158
      port: 3306
      weight: 30
```

설정 추가하고 적용하면 클러스터가 뜨면서 복제를 시작한다.

새로 올라온 디비를 확인해보면 복제가 되는것을 알수 있다.

