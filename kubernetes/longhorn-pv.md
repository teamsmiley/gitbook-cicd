# longhorn - pv

노드장비를 storage로 사용할수 있다.

persistance volume을 kubernetes 가 설치된 노드를 pv로 사용한다.

pod가 1번 노드에서 뜨면 1번 노드에 pv를 붙여주는거 같음

복제본도 만들어주고 snapshot도 해주고 그런다.

{% embed url="https://longhorn.io/" caption="" %}

## prerequisites

ubuntu20은 nfs-commons만 추가 설치해야한다.

```bash
apt install nfs-common -y
```

전체 노드에 설치를 한다.

## longhorn 설치

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm search repo longhorn
helm install longhorn longhorn/longhorn --namespace longhorn-system
```

LoadBalancer타입으로 설치하려면 다음처럼 하면된다.

```bash
helm install longhorn longhorn/longhorn --set service.ui.type=LoadBalancer -n longhorn-system --create-namespace

helm list -n longhorn-system

kcn longhorn-system

k get pod
```

프론트웹으로 접속해서 volume을 하나 만들어보자.

![](../.gitbook/assets/2021-08-10-18-52-04.png)

```bash
k get volumes

NAME   STATE      ROBUSTNESS   SCHEDULED   SIZE          NODE   AGE
test   detached   unknown      True        21474836480          7m57s

k get crd | grep longhorn

backingimagemanagers.longhorn.io                      2021-08-11T01:07:42Z
backingimages.longhorn.io                             2021-08-11T01:07:42Z
engineimages.longhorn.io                              2021-08-11T01:07:42Z
engines.longhorn.io                                   2021-08-11T01:07:42Z
instancemanagers.longhorn.io                          2021-08-11T01:07:42Z
nodes.longhorn.io                                     2021-08-11T01:07:42Z
replicas.longhorn.io                                  2021-08-11T01:07:42Z
settings.longhorn.io                                  2021-08-11T01:07:42Z
sharemanagers.longhorn.io                             2021-08-11T01:07:42Z
volumes.longhorn.io                                   2021-08-11T01:07:42Z
```

volume이라는 crd를 제공함..기타 다른것도 제공.

longhorn이라는 스토리지 클라스 제공 디폴트로 세팅되어 있다.

```bash
# k get sc
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
longhorn (default)   driver.longhorn.io   Delete          Immediate           true                   33m
```

pv와 pvc가 자동으로 생성되는 준비가 됬다.

## ingress - dashboard

```text
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn
  namespace: longhorn-system
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: longhorn.yourdoman.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: longhorn-frontend
                port:
                  number: 80
```

[http://longhorn.yourdoman.com](http://longhorn.yourdoman.com)으로 확인해보면 dashboard가 사용이 가능하다.

## pvc 생성

```text
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

mypvc.yml

```bash
k apply -f mypvc.yml
```

longhorn dashboard에서 volume이 생긴걸 확인할수 있다.

![](../.gitbook/assets/2021-08-13-17-17-07.png)

detached 상태이다.

## pod에서 사용

vi pod.yml

```text
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
        - mountPath: '/var/www/html'
          name: mypod
  volumes:
    - name: mypod
      persistentVolumeClaim:
        claimName: mypvc
```

```bash
k apply -f pod.yml
```

![](../.gitbook/assets/2021-08-13-17-18-12.png)

조금 기다리면 volume이 attached가 된걸 볼수가 있다.

![](../.gitbook/assets/2021-08-13-17-18-29.png)

볼륨 name을 눌러서 들어가보면 자세한 내용이 나온다.

![](../.gitbook/assets/2021-08-13-17-19-13.png)

backup도 할수 있고 snapshot도 할수 있다.

replica가 기본값이 3 이므로 3개의 리플리카를 만들어주고 제일먼저 같은 노드에 잇는 pv를 pod에 붙여준다.

이 값을 줄이려면

![](../.gitbook/assets/2021-08-13-17-20-15.png)

![](../.gitbook/assets/2021-08-13-17-20-36.png)

이 버튼을 이용하면된다.

1 개로 줄이면 줄어드는것을 볼수 있다.

![](../.gitbook/assets/2021-08-13-17-22-53.png)

현재 노드가 6개이므로 6개로 늘려주면 모든 노드에 데이터가 다 쌓이므로 파드가 다른 노드로 움직여도 문제가 없는듯 보인다.

메뉴중에 update data locality를 눌러보자.

![](../.gitbook/assets/2021-08-13-17-21-52.png)

![](../.gitbook/assets/2021-08-13-17-22-03.png)

best effort로 하면 pod가 있는 노드에서 pvc를 붙여주는듯 보인다.

현재 node05번에 pvc가 있다. ![](../.gitbook/assets/2021-08-13-17-24-58.png)

pod도 5번에 있는것을 확인할수 있다.

![](../.gitbook/assets/2021-08-13-17-25-54.png)

## snapshot

![](../.gitbook/assets/2021-08-13-17-32-30.png)

![](../.gitbook/assets/2021-08-13-17-32-42.png)

간단히 snapshot이 됬다. 메인으로 사용되지 않는 replica에서 스냅샷이 이루어지는지는 확인하지 못햇다.

![](../.gitbook/assets/2021-08-13-17-34-00.png)

스냅샷 리스트가 보인다.

복구를 해보자.

스냅샷을 고르면 다음처럼 나온다.

![](../.gitbook/assets/2021-08-13-17-35-08.png)

revert가 회색인것을 알수있다. pod가 attached되어 있어서 생기는 문제 일단 pod를 내리고 다시 확인해보자.

```bash
kubectl delete pod mypod
```

![](../.gitbook/assets/2021-08-13-17-36-58.png)

detached로 바뀐 볼륨

detached된 볼륨을 선택하고 attach를 누르자.

![](../.gitbook/assets/2021-08-13-17-40-30.png)

mainterance mode로 들어가서 작업하자. mainterance 를 꼭 체크해준다.

![](../.gitbook/assets/2021-08-13-17-39-47.png)

health로 바뀐다.

![](../.gitbook/assets/2021-08-13-17-42-02.png)

이름을 클릭하고 내부로 들어가보면 snapshot이 보인다. 클릭하면 revert가 활성화 된다. 누르자.

![](../.gitbook/assets/2021-08-13-17-43-04.png)

revert가 다된후에 다시 volume을 detach한다.

![](../.gitbook/assets/2021-08-13-17-44-44.png)

detached된 볼륨

![](../.gitbook/assets/2021-08-13-17-45-14.png)

이제 pod를 다시 생성해보자.

```bash
k apply -f mypod.yml
```

![](../.gitbook/assets/2021-08-13-17-47-51.png)

다시 pod에 attched됫다.

스냅샷 버전으로 돌아온것을 확인할수 있다.

근데 왜 같은 volume으로 붙지? 아 pvc가 지워지지 않고 유지되고 있어서 그렇지.

## recurring snapshot and backup

![](../.gitbook/assets/2021-08-13-17-50-21.png)

여기서 처리하면된다.

![](../.gitbook/assets/2021-08-13-17-50-42.png)

![](../.gitbook/assets/2021-08-13-17-51-19.png)

스냅샷과 백업 둘다 할수 있다.

## 기본 옵션 변경

메뉴중 setting이라는곳에 모든 기본 옵션이 들어가 있다.

![](../.gitbook/assets/2021-08-13-17-27-15.png)

helm설치시 설정해주어도 된다.

## backup

리플리카가 있기는 하지만 클러스터 내부에 있으므로 외부로 백업을 하는것도 가능하다.

s3에 백업을 하자. s3권한문제와 secret key부분은 각자 알아서 처리해두고 진행

bucket을 만든다.

![](../.gitbook/assets/2021-08-13-18-06-35.png)

aws 접속정보가 있는 secret를 만들고 적용한다.

```text
apiVersion: v1
kind: Secret
metadata:
  name: backup-s3
  namespace: longhorn-system
type: Opaque
data:
  AWS_ACCESS_KEY_ID: AAAA=
  AWS_SECRET_ACCESS_KEY: AAAA==
```

```bash
k apply -f backup-s3.yml
k get secret -n longhorn-system
```

![](../.gitbook/assets/2021-08-13-18-01-50.png)

longhorn 세팅 페이지에서 설정을 하자.

![](../.gitbook/assets/2021-08-13-18-03-35.png)

`s3://bucket-name@region/path/`

save하자.

volume에 들어가서 백업 버튼을 눌러보자.

![](../.gitbook/assets/2021-08-13-18-12-42.png)

조금 기다리면 백업이 됬다고 표시된다.

s3에서 확인해보면 다음과 같이 업로드된것이 보인다.

![](../.gitbook/assets/2021-08-13-18-13-45.png)

매일 스케줄을 걸어두면 편할듯 보인다.

![](../.gitbook/assets/2021-08-13-18-18-16.png)

특정 갯수 이상은 s3에서 지워준다.

시간이 되니 자동으로 업로드해주고 백업 생성해준다.

![](../.gitbook/assets/2021-08-13-18-20-45.png)

