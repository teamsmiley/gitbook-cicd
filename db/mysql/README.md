# Mysql

## Percona Xtradb Cluster

mysql관련해서는 Percona Xtradb Cluster를 추천하며 줄여서 pxc라고 부른다.

직접 노드에 설치해도 되며 kubernetes에 operator를 제공하여 편하게 사용할수도 있다.

다만 쿠버네티스를 사용할 경우는 longhorn등 쿠베 클러스터내부에 하드를 스토리지로 사용하는 솔류션을 같이 쓰기를 추천한다.

그렇게 되면 pod가 뜬 곳에서 로컬하드를 이용하므로 빠른 io가 보장된다.

mysql과 호환된다.

