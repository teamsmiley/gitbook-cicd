# curity identity server

identity server중에 하나로 새로 배우게 되었다.

## 회원가입후 라이센스 받기

https://developer.curity.io/

커뮤니티 에디션으로 라이센스를 받아서 다운로드해두자.

## arogocd / helm으로 설치

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: curity
  namespace: argocd
spec:
  destination:
    name: ''
    namespace: curity
    server: 'https://kubernetes.default.svc'
  source:
    path: ''
    repoURL: 'https://curityio.github.io/idsvr-helm/'
    targetRevision: 0.9.26
    chart: idsvr
    helm:
      parameters:
        - name: curity.adminUiHttp
          value: 'true'
        - name: curity.config.uiEnabled
          value: 'true'
        - name: curity.config.password
          value: kimchi66
        - name: curity.admin.logging.stdout
          value: 'true'
        - name: ingress.enabled
          value: 'true'
        - name: ingress.runtime.host
          value: curity.xgridcolo.com
        - name: ingress.admin.host
          value: admin.curity.xgridcolo.com
        - name: networkpolicy.enabled
          value: 'false'
        - name: replicaCount
          value: '3'
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## ingress로 접근하는법

`https://admin.curity.xgridcolo.com/admin/`

## port forwarding으로 접근하는법

```sh
export POD_NAME=$(kubectl get pods -l "role=curity-idsvr-admin" -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward $POD_NAME 6749:6749
```

`http://localhost/admin/`

port forwarding으로는 접근이 되고 ingress로 안되면 helm 옵션중에 network policy를 끄고 실행해봐라 그럼 될것이다.

## Run Basic Setup

설정해둔 url로 접근하면 다음 화면이 나온다.

![](./images/2021-08-31-06-34-51.png)

설정해둔 비번으로 로그인

![](./images/2021-08-31-06-31-58.png)

Run Basic Setup 클릭

![](./images/2021-08-31-06-32-42.png)

커뮤니티 라이센스를 발급받아서 업로드 해주자.

![](./images/2021-08-31-06-33-20.png)

next 클릭

![](./images/2021-08-31-06-33-41.png)

일단 기본값으로 모두 next를 누르면 된다.

![](./images/2021-08-31-06-35-42.png)

![](./images/2021-08-31-06-35-53.png)

![](./images/2021-08-31-06-36-03.png)

![](./images/2021-08-31-06-36-14.png)

![](./images/2021-08-31-06-36-29.png)

![](./images/2021-08-31-06-36-40.png)

![](./images/2021-08-31-06-36-51.png)

마지막에 커밋버튼을 클릭하자.

curity는 유저데이터등은 db에 저장하지만 이 설정파일등은 xml로 로컬에 저장하는것같다.

다 로딩이 되면

![](./images/2021-08-31-06-37-46.png)

admin 하나와 runtime 모듈 3개가 올라와 있다.

![](./images/2021-08-31-06-38-19.png)

## url change

baseurl을 수정해주자.

![](./images/2021-08-31-06-40-26.png)

general 메뉴에서도 수정

![](./images/2021-08-31-06-41-21.png)

변경할때마다 commit을 해야한다.

## jdbc

curity는 jdbc드라이버를 포함하고 있지 않다. 오라클 라이센스때문에 직접 다운받아서 컨테이너에 넣어줘야한다.

```sh
kubectl cp ~/Downloads/mysql-connector-java-8.0.26.jar -n curity $(kubectl get pods -l "role=curity-idsvr-admin" -o jsonpath="{.items[0].metadata.name}"):/opt/idsvr/lib/plugins/data.access.jdbc/
```

//todo
이부분은 나중에 좀더 다듬어야할듯. 매번 컨테이너 올라올때마다 넣어줄수는 없으니.

이제 jdbc connection string을 적어주자.

<https://curity.io/docs/idsvr/latest/system-admin-guide/data-sources/jdbc.html?highlight=session#mysql-and-mariadb>

![](./images/2021-08-31-07-04-35.png)

![](./images/2021-08-31-07-11-15.png)

create

```text
jdbc:mysql://MYSQL_HOST:3306/se_curity_store?useSSL=false
```

설정하자.

![](./images/2021-08-31-07-07-50.png)

![](./images/2021-08-31-07-09-33.png)

![](./images/2021-08-31-07-10-16.png)

추가 완료

테이블을 생성해줘야한다.

스크립트는 어드민 컨테이너에 있다.

![](./images/2021-08-31-07-13-40.png)

가지고와서 디비에 적용해주자.

```sql
create database se_curity_store;
```

```sh
kubectl cp -n curity $(kubectl get pods -l "role=curity-idsvr-admin" -o jsonpath="{.items[0].metadata.name}"):/opt/idsvr/etc/mysql-create_database.sql ~/Downloads/mysql-create_database.sql
```

![](./images/2021-08-31-07-19-38.png)

이상하게 에러가 나서 Linked Accounts 앞까지만 먼저 실행하고 완료후 뒤 스크립트를 실행하였다.

commit

디비까지 완료

## 현재까지 구조

![](./images/2021-08-31-07-31-22.png)

## custom image

Dockerfile을 만들어서 커스터마이즈하자 jdbc 파일을 복사해야함.

나중에 쓸려고 git도 설치가 완료가 되야함.

```sh
cat > Dockerfile <<EOF
FROM curity.azurecr.io/curity/idsvr:6.4.1

COPY mysql-connector-java-8.0.26.jar /opt/idsvr/lib/plugins/data.access.jdbc/

USER root

RUN apt update -y
RUN apt install git curl -y

USER idsvr:idsvr
EOF

docker build . -t curity-custom

docker run -it -e PASSWORD=kimchi66 -p 6749:6749 -p 8443:8443 --name curity curity-custom
```

잘 실행되나 보고 jdbc driver 있는지 보고 git/curl잘되는지 확인하면 된다.

## 설정파일 백업

user data는 외부 디비에 저장되므로 상관없지만 설정파일은
pod가 옮겨지면 모두 없어진다.

admin에 xml로 생성이 되니 이걸 백업 받아야한다.

설정파일을 저장할 깃허브 repo를 만들자. 그리고 PAT(personal access token)을 생성 저장해두자.

curity가 commit hooks를 지원한다.

컨테이너에 /opt/idsvr/usr/bin/post-commit-cli-scripts/ 에 스트립트를 넣어주면 실행을 한다.

<https://curity.io/docs/idsvr/latest/configuration-guide/commit-hooks.html#commit-hook-cli-scripts>

custom image를 만들때 이 파일을 아에 넣어주면 좋을거같다.

```sh
cat > full-backup.cli <<EOF
show configuration | display xml | save /tmp/config-backup.\${EPOCH_TIMESTAMP}.xml

EOF
```

vi Dockerfile

```text
FROM curity.azurecr.io/curity/idsvr:6.4.1

COPY mysql-connector-java-8.0.26.jar /opt/idsvr/lib/plugins/data.access.jdbc/

COPY full-backup.cli /opt/idsvr/usr/bin/post-commit-cli-scripts/

USER root

RUN apt update -y
RUN apt install git -y
RUN apt install curl -y

USER idsvr:idsvr
```

이제 이 도커파일을
이제 웹사이트에서 뭔가를 바구고 commit을 해보자.

컨테이너에 /tmp에 파일이 저장됫는지 확인

![](./images/2021-08-31-07-55-29.png)

생성되었다.

이걸 외부로 보내려면?

github에 repo를 만들자.