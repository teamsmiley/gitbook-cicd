# manage maas

cli를 설치한후 다음처럼 하자.

### Create an administrator <a id="heading--create-an-administrator"></a>

PROFILE=admin

EMAIL\_ADDRESS = "brian@xgridcolo.com"

MAAS\_URL=[http://localhost:5240/MAAS/api/2.0](http://localhost:5240/MAAS/api/2.0)

sudo maas createadmin --username=$PROFILE --email=$EMAIL\_ADDRESS

웹사이트로 로그인후 user를 클릭한다. apikey를 복사한다.

maas login $PROFILE $MAAS\_URL

api key를 복사해둔것을 붙여넣기 한다.

![](../.gitbook/assets/image.png)

로그인이 된다.

maas $PROFILE --help

maas $PROFILE tag --help

maas $PROFILE tags --help



