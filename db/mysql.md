# mysql 8.0

## 로그인 방식의 변경

요즘 mysql에서 로그인방식이 변경되었다고한다.

오랜만에 설치햇다가 맨붕에 빠짐.

정리해보면 두가지 방식이 있다.

- unix_socket 인증
- mysql_native_password 인증

### unix_socket 인증

유닉스 계열 운영체제의 사용자 계정과 mysql의 사용자 계정을 일치시키는 인증 방식

ubuntu 20.04에서 설치

```sh
sudo apt update -y
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

이제 접속해보자.

```sh
sudo mysql
```

로그인이 된다.

mysql_secure_installation에서 설정한 패스워드가 아니라도 로그인이 되버린다.

유닉스 계열 운영체제의 root 계정(또는 root 계정의 권한을 가진 계정)은 mysql의 root 계정을 소유하여 관리할 수 있도록 한 것입니다.

이 unix_socket 인증을 사용하면 유닉스 계열 운영 체제의 루트 사용자가 소유 및 실행한 프로세스에서 mysql의 콘솔에 로그인하는 경우 소켓 시스템 변수에 정의된 로컬 Unix 소켓 파일을 통해 암호 입력 없이 로그인 할 수 있습니다.

### mysql_native_password 인증

mysql_native_password 인증 방식은 전통적으로 쓰이던 방식으로 로그인 시 아래와 같이 계정명과 암호를 입력하여 로그인하는 방식입니다.

```sh
mysql -u root -p
```

정리하면 더이상 root 비번을 넣을 필요는 없는듯 보이고 유저를 만들어서 사용하면 될것같다.

서버에 접속되면 서버에서는 `sudo mysql`하면 루트로 로그인이 된다.
