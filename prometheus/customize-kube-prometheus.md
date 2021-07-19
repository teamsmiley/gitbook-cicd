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
