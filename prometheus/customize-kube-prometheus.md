# Customize kube-prometheus

소스코드를 받은걸 컴파일을 하면 자기가 원하는 모양으로 만들어둘두 있다고 한다.

go를 사용해야한다. jsonnet을 사용한다.

```bash
brew install go

export PATH=$PATH:$(go env GOPATH)/bin
export GOPATH=$(go env GOPATH)

# get jb compile tool
go get -u github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb

mkdir core/prometheus
cd core/prometheus

jb init

jb install github.com/prometheus-operator/kube-prometheus/jsonnet/kube-prometheus@release-0.8

wget https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/release-0.8/example.jsonnet -O example.jsonnet

wget https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/release-0.8/build.sh -O build.sh

chmod 700 build.sh

#jb update
#go get github.com/google/go-jsonnet/cmd/jsonnet
#go get github.com/brancz/gojsontoyaml
#./build.sh

docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh example.jsonnet
```

grafana/prometheus/alertmanager svc가 현재는 clusterip 인데 node port로 변경해보자.

`kube-prometheus/addons/node-ports.libsonnet` 이부분만 주석해제 해주면된다.

```jsonnet
local kp =
  (import 'kube-prometheus/main.libsonnet') +
  (import 'kube-prometheus/addons/node-ports.libsonnet')
  {

```

다시 빌드하고 커밋하면된다.

```sh
docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh example.jsonnet
```

커밋후 svc를 확인해보면 서비스 타입이 노드포트로 변경되는걸 알수 있다.

이제 다시 지워고 다시 빌드 커밋 푸시 하면 원래대로 돌아오는것을 알수 있다.

## ingress에 서비스를 오픈

<https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/exposing-prometheus-alertmanager-grafana-ingress.md>

참고해서 봐도 되고 다음 코드도 같이 보면 좋다.

<https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/ingress.jsonnet>

example.jsonnet을 위 파일처럼 수정후 domain을 변경해주면된다.

auth라는 파일을 참조하는것을 알수 있다.

이걸 만들기 위해서는 다음과 같이 한다.

```bash
sudo apt install apache2-utils
htpasswd -c auth admin
> password를 넣는다. 엔터
```

auth라는 파일이 생겼다. 내용을 복사하여 example.jsonnet파일과 같은 디렉토리에 복사해서 넣어준다.

도메인으로 접근하면 basic login화면이 나오고 생성해준 id 비번을 넣으면 로그인이 된다.

하고보면 grafana는 id/password를 두번 넣어야하는 문제가 생기는데?

## etcd 모니터링

```sh
ssh master01

# Copy etcd CA cert from etcd server "/etc/ssl/etcd/ssl/ca.pem"
sudo cp /etc/ssl/etcd/ssl/ca.pem /home/ubuntu/
# Copy etcd CA cert from etcd server "/etc/ssl/etcd/ssl/ca-key.pem"
sudo cp /etc/ssl/etcd/ssl/ca-key.pem /home/ubuntu/
cd /home/ubuntu/

apt install golang-cfssl

cat client.json
```

```json
{
  "CN": "etcd-ca",
  "hosts": [""],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{}]
}
```

```sh
# Generate client certificate
cfssl gencert -ca ca.pem -ca-key ca-key.pem client.json | cfssljson -bare etcd-client

chmod 755 *.pem
```

관련 파일이 만들어진다. 전부 로컬로 가져온다.

```bash
scp master01.c3:~/ca.pem ~/Desktop/GitHub/argocd-c3/core/prometheus/etcd
scp master01.c3:~/etcd-client-key.pem ~/Desktop/GitHub/argocd-c3/core/prometheus/etcd
scp master01.c3:~/etcd-client.pem ~/Desktop/GitHub/argocd-c3/core/prometheus/etcd
```

jsonnet 설정

<https://github.com/prometheus-operator/kube-prometheus/blob/main/examples/etcd.jsonnet>

여기 참고하면된다.

아이피는 사용하는 아이피 전부 넣어주면되고 서버이름은 빈칸으로 해도 된다. insecureSkipVerify 는 false로

```jsonnet
etcd+: {
        ips: ['172.16.3.11', '172.16.3.12', '172.16.3.13'],
        clientCA: importstr 'etcd/ca.pem',
        clientKey: importstr 'etcd/etcd-client-key.pem',
        clientCert: importstr 'etcd/etcd-client.pem',
        //serverName: 'etcd.kube-system.svc.cluster.local',
        serverName: '',

        insecureSkipVerify: true,
      },
```

빌드하고 커밋 푸시해보자.
