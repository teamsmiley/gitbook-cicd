# Cross Site Replication

두개의 쿠버네티스에서 디비를 리플리케이션 할수 있다.

[https://www.percona.com/doc/kubernetes-operator-for-pxc/replication.html](https://www.percona.com/doc/kubernetes-operator-for-pxc/replication.html)

![](../../.gitbook/assets/2021-08-22-23-54-23.png)

## 주의

* Percona XtraDB Cluster 8.0.22+ 이상을 설치해야 한다.
* haproxy를 사용하면 잘되는데 proxy sql을 사용하면 안된다.
* 복제시 사용할 비번이 같아야한다. secret.yml에 같은 비번으로 설정해주면 된다.

## 고려사항

* pitr을 replica 클러스터에서 할것인가?
* 백업은 어느 클러스터에서 할것인가?

source cluster에서 하는것으로 결정 왜냐면 마스터가 아무래도 최신이기 때문이다.

전체 백업은 source cluster에서 하므로 리플리카에서는 안하는것으로한다.

## source cluster 구성

{% code title="cr.yaml" %}
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
{% endcode %}

체널을 설정하고 isSource를 true로 설정한다.

load balance를 이용하여 svc를 통해 외부에 오픈한다.

`kubectl get services -l "app.kubernetes.io/instance=CLUSTER_NAME"` 서비스를 확인하자. 아이피를 확인하자.

## replica cluster

복제를 받을 클러스터에서 설정하자.

위에서 로드발란스로 오픈한 아이피를 사용한다.

{% code title="cr.yaml" %}
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
{% endcode %}

설정 추가하고 적용하면 클러스터가 뜨면서 복제를 시작한다.

새로 올라온 디비를 확인해보면 복제가 되는것을 알수 있다.

## 추가

이제 source cluster가 문제가 생겨서 지워지면 replica cluster를 기본으로 돌려야할건데 어떻게 하지?

