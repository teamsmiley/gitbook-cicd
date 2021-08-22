# external-dns

ingress에 적용한 도메인을 도메인 서버에 자동으로 만들어준다.

도메인 서버에 인증부분을 해결해야하고 api를 쓸수 있어야한다. 

firewall에서 아이피 매핑을 쓰는경우는 ? 문제가 생긴다 클러스터 인그레스 아이피를 서버로 보내버리기 때문이다.

이럴때는 다음  인그레스에서 다음 코드를 사용 외부아이피를 고정할수 있다.

```text
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: default
  annotations:
    external-dns.alpha.kubernetes.io/target: "100.16.116.99"
```



