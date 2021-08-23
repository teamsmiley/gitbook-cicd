# GTID

Global Transaction Identifier

GTID를 사용하게 되면, 각각의 트렌젝션들은 고유한 전역식별자를 갖게 된다.

master 서버에서 수행된(commited) 트랜젝션들이 slave 서버(들)에 적용되어 지는 것에대한 추적 또한 쉽다.

새로운 Slave의 추가 구성이나, 신규 Master로 Failover 할 때에도, binary log files와 postion의 확인은 중요하지 않고, 작업 역시 간단해 진다.

GTID 기반 복제는 완전히 트랜젝션 기반이기 때문에, Master와 Slave 간의 일관성 확인이 간단하다.

GTID 기반 복제는 Statement-based replication과 row-based replication을 모두 지원한다.



출처: https://minsugamin.tistory.com/entry/MySQL-57-GTID [즐거운 인생~]