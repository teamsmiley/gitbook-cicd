# submodule

## submodule 등록

git에서 다른 git을 하위폴더에 가져와서 같이 사용하는것

```bash
git clone main_repo_url

git submodule add git@github.com:prometheus-operator/kube-prometheus.git
cd kube-prometheus
git checkout tags/v0.8.0
```

## clone

클론을 처음 받을때 submodule까지 받을수 있다.

```sh
git clone --recursive git@git://github.com/foo/bar.git
```

## 클론을 미리 받은경우

새로 체크아웃 받는경우 submodule은 다운로드 되지 않는다. 따로 관리해야한다.

```bash
git clone git://github.com/foo/bar.git
cd bar
git submodule update --init --recursive
```

## tag를 유지

checkout을 tag로 해두면 이 버전이 유지된다.


## submodule 삭제

```bash
submodule_path=kube-prometheus

git rm --cached ${submodule_path}
```

.gitmodules 에서 원하는 git submodule을 삭제한다.

폴더를 삭제한다.

```bash
rm -rf ${submodule_path}
rm -rf .git/modules/${submodule_path}
```

## 커맨드로만 하는 방법

```bash
submodule_path=kube-prometheus

git submodule deinit -f ${submodule_path}
git rm ${submodule_path}
# Note: submodule_path (no trailing slash)
git rm --cached ${submodule_path}
rm -rf .git/modules/${submodule_path}
```

## todo

subtree와 차이점은 무엇일가?

