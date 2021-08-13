# percona extra db  cluster operator

mysql을 설치해주고 복제해주고 모든걸 자동으로 해주는것이 목표인다. 

pv/pvc가 없이 tempdir 이나 hostpath로도 테스트는 가능하나 백업/복구 등에 문제가 있다. 

현재는 모든 스크립트가 pv/pvc가 있는것을 가정하는것 같다. 

그러므로 앞에 설치한 longhorn을 설치를 해두고 작업하기를 바란다

s3에 백업도 해준다. 그걸 가지고 와서 복구도 해준다.

proxysql도 추가해서 사용이 가능하다.  그러면 문제가 생기는 노드는 서비스에서 자동을 제거할수 있다.

pmm이라고 perconam monitering and management라는 패키지도 제공한다. 프로메테우스와 그라파나를 이용해서 현재 클러스터 메트릭을 모아서 볼수잇게 해준다 

pmm proxysql s3backup이 모두 필요하다고 보고 모두 설치해서 진행할 예정



