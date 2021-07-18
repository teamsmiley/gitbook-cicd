# Customize kube-prometheus

소스코드를 받은걸 컴파일을 하면 자기가 원하는 모양으로 만들어둘두 있다고 한다.

go를 사용해야한다. jsonnet을 사용한다.

```bash
brew install go

export PATH=$PATH:$(go env GOPATH)/bin
export GOPATH=$(go env GOPATH)

# get jb compile tool
go get -u github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb

cd core/kube-prometheus

jb update

go get github.com/google/go-jsonnet/cmd/jsonnet
go get github.com/brancz/gojsontoyaml

chmod 700 build.sh

./build.sh

```

## docker

```sh
docker run --rm -v $(pwd):$(pwd) --workdir $(pwd) quay.io/coreos/jsonnet-ci ./build.sh example.jsonnet
```

