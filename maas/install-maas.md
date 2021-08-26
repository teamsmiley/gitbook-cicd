# maas 설치

<https://maas.io/>

## os install

install ubuntu 20.04 LTS with CD

```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

## ipmi 설정

```sh
sudo apt install ipmitool -y
ip=10.1.4.11
sudo ipmitool lan set 1 ipsrc static
sudo ipmitool lan set 1 ipaddr ${ip}
sudo ipmitool lan set 1 netmask 255.255.255.0
sudo ipmitool lan set 1 defgw ipaddr 10.1.4.1
sudo ipmitool lan set 1 arp respond on
```

## ip setting

```sh
sudo vi /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    eno1:
      addresses:
        - 10.1.5.11/24
      gateway4: 10.1.5.1
      nameservers:
        addresses:
          - 4.2.2.2
        search: []
    eno2:
      dhcp4: true
  version: 2
```

```sh
sudo netplan apply
ifconfig #확인
```

## snap and postgresql install

```sh
sudo snap install --channel=3.0/stable maas
sudo apt install -y postgresql

MAAS_DBUSER=XXXXXXX
MAAS_DBPASS=XXXXXXX
MAAS_DBNAME=maas

sudo -u postgres psql -c "CREATE USER \"$MAAS_DBUSER\" WITH ENCRYPTED PASSWORD '$MAAS_DBPASS'"

sudo -u postgres createdb -O "$MAAS_DBUSER" "$MAAS_DBNAME"
```

```sh
sudo vi /etc/postgresql/12/main/pg_hba.conf
```

```bash
# host    $MAAS_DBNAME    $MAAS_DBUSER    0/0     md5
host      maas            XXXX           0/0     md5
```

![](./images/2021-08-24-20-29-07.png)

```bash
sudo maas init region+rack --database-uri "postgres://$MAAS_DBUSER:$MAAS_DBPASS@localhost/$MAAS_DBNAME"

#sudo maas init region+rack --database-uri "postgres://$MAAS_DBUSER:$MAAS_DBPASS@$HOSTNAME/$MAAS_DBNAME"
MASS URL : (just enter)
```

결과

```txt
MAAS URL [default=http://10.1.5.11:5240/MAAS]:
MAAS has been set up.

If you want to configure external authentication or use
MAAS with Canonical RBAC, please run

  sudo maas configauth

To create admins when not using external authentication, run

  sudo maas createadmin
```

```bash
sudo maas status

> bind9                            RUNNING   pid 8142, uptime 0:02:21
> dhcpd                            STOPPED   Not started
> dhcpd6                           STOPPED   Not started
> http                             RUNNING   pid 8411, uptime 0:00:46
> ntp                              RUNNING   pid 8327, uptime 0:00:51
> proxy                            RUNNING   pid 8486, uptime 0:00:39
> rackd                            RUNNING   pid 8145, uptime 0:02:21
> regiond                          RUNNING   pid 8146, uptime 0:02:21
> syslog                           RUNNING   pid 8324, uptime 0:00:51
```

## add admin

```bash
sudo maas createadmin --username=admin --email=brian@xgridcolo.com

> YourPassword

Import SSH keys [] (lp:user-id or gh:user-id): (just enter)
```

웹 브라우져

<http://10.1.5.11:5240/MAAS/> (대소문자 주의)

![](./images/2021-08-24-20-32-22.png)

![](./images/2021-08-24-20-32-44.png)

continue

## upload ssh key import

초기 접속 ssh key를 설정한다. 중요하다.

![](./images/2021-08-24-20-33-48.png)

![](./images/2021-08-24-20-34-37.png)

continue

```bash
cat ~/.ssh/id_rsa.pub
>  ssh-rsa AAxxx0RVSJOdOBSeO7e
```

## dhcp를 enable

subnet >> click

untagged click

![](./images/2021-08-25-07-52-28.png)

enable dhcp 클릭

![](./images/2021-08-25-07-53-42.png)

![](./images/2021-08-25-07-54-01.png)

```bash
sudo maas status

bind9                            RUNNING   pid 19886, uptime 0:09:28
dhcpd                            RUNNING   pid 20664, uptime 0:00:49
dhcpd6                           STOPPED   Not started
http                             RUNNING   pid 20198, uptime 0:07:34
ntp                              RUNNING   pid 20085, uptime 0:07:43
proxy                            RUNNING   pid 20602, uptime 0:02:28
rackd                            RUNNING   pid 19889, uptime 0:09:28
regiond                          RUNNING   pid 19890, uptime 0:09:28
syslog                           RUNNING   pid 20084, uptime 0:07:43
```

dhcp가 실행중임을 확인할수 있다.

## image 다운로드

http://10.1.5.11:5240/MAAS/l/images

sync임을 확인할수 있다.

## 새노드 설치

노드 부팅 순서를 pxe를 1번으로 해두면 자동으로 maas에서 이미지를 받아서 설치한다. 웹 화면에서도 자동 등록이 된다.

![](./images/2021-08-25-08-19-00.png)

자동으로 등록되며 commissioning까지 된다. commsioning이 실패하면 new 로 되고 통과하면 ready가 된다.

상태를 설명하면 아래와 같다.

```text
new -> commissioning -> ready -> deploy
```

new 상태로 간다.

![](./images/2021-08-25-08-22-09.png)

--> commisioning을 추가로 해보자.

![](./images/2021-08-25-08-59-03.png)

--> ready상태임

이름 바꾸고 ip를 지정을 해보자.

![](./images/2021-08-25-08-26-07.png)

![](./images/2021-08-25-08-27-00.png)

![](./images/2021-08-25-08-28-17.png)

## install os

이제 deploy를 해보자.

장비를 선택하고 deploy를 누르면 된다.

![](./images/2021-08-25-09-00-43.png)

bearmetal장비이므로 kvm도 같이 설치가 되게 해두었다.

![](./images/2021-08-25-09-01-48.png)

디플로이 해보자.

![](./images/2021-08-25-09-23-24.png)

잘 설치 되었다.

## vm도 설치해보자.

![](./images/2021-08-25-09-27-29.png)

노드 이름을 누르고 들어가서 compose를 눌른다.

![](./images/2021-08-25-09-29-57.png)

memory , core , harddisk 설정을 해보자.

생성하자.

machine 메뉴로 가보자.

새로 생성된 vm이 새 장비로 보이고 commisioning이 시작되었다.

![](./images/2021-08-25-09-31-25.png)

ready상태로 바뀌고 디플로이를 대기한다.

![](./images/2021-08-25-09-34-34.png)

이제 os를 deploy 하면 된다.

완료후 접속해보자 ubuntu유저와 초기에 등록한 ssh key로 접속이 가능하다.
