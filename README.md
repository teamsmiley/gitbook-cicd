# Modern DEVOPS

내가 생각하기에 사용하기 좋은 devops환경 구축 정리 해보았다.

1. source repository
   1. github
   2. gitlab
2. ci/cd
   1. github action
   2. gitlab cicd
3. 배포
   1. cicd로 docker image 생성
4. docker image registry 에 저장
   1. docker hub
   2. private registry docker
   3. gitalb container registry
   4. github package
   5. ECR (aws)
5. kubernetes 에서 도커이미지들 배포
   1. bearmetal
   2. EKS (aws)
6. argocd kubernetes 관리
   1. gitops
   2. notification-slack
7. kubernetes 모니터링
   1. Prometheus
   1. Grafana
8. 관련 소스코드
   - [https://github.com/teamsmiley/devops-senima-argocd](https://github.com/teamsmiley/devops-senima-argocd)
   - [https://github.com/teamsmiley/devops-semina-sourcecode](https://github.com/teamsmiley/devops-semina-sourcecode)
