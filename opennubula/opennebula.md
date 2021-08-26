# opennebula

vm을 관리해주는 오픈소스입니다.

<https://docs.opennebula.io/6.0/installation_and_configuration/frontend_installation/index.html>

## install db mysql

```sh
sudo apt update -y
sudo apt install mysql-server -y
sudo mysql_secure_installation

sudo mysql
CREATE DATABASE opennebula;
CREATE USER 'oneadmin' IDENTIFIED BY 'your-password';
GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
flush privileges;
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

이제 접속해보자.

```sh
mysql -u oneadmin -p
```

## install opennebula

```sh
sudo apt update -y
sudo apt  install gnupg wget apt-transport-https -y

sudo -i

wget -q -O- https://downloads.opennebula.io/repo/repo.key | apt-key add -

echo "deb https://downloads.opennebula.io/repo/6.0/Ubuntu/20.04 stable opennebula" > /etc/apt/sources.list.d/opennebula.list

exit

sudo apt update -y

sudo apt install opennebula opennebula-sunstone opennebula-fireedge opennebula-gate opennebula-flow opennebula-provision -y

# oneadmin유저가 자동으로 생성됨.

sudo vi /etc/one/oned.conf

# Sample configuration for PostgreSQL
# DB = [ BACKEND = "mysql",
#        SERVER  = "localhost",
#        PORT    = 0,
#        USER    = "oneadmin",
#        PASSWD  = "<thepassword>",
#        DB_NAME = "opennebula",
#        CONNECTIONS = 25,
#        COMPARE_BINARY = "no" ]

sudo -u oneadmin /bin/sh

echo 'oneadmin:changeme123' > /var/lib/one/.one/one_auth

exit

sudo ufw allow proto tcp from any to any port 9869

sudo systemctl start opennebula opennebula-sunstone

sudo systemctl enable opennebula opennebula-sunstone

sudo systemctl status opennebula

sudo systemctl status opennebula-sunstone

# 확인
sudo -i
oneuser show

> USER 0 INFORMATION
> ID              : 0
> NAME            : oneadmin
> GROUP           : oneadmin
> PASSWORD        : > 39fb427b99fad4b6b2f4547c07f509af87332f70664e1c7c09467cb5829eb833
> AUTH_DRIVER     : core
> ENABLED         : Yes
>
> TOKENS
>
> USER TEMPLATE
> TOKEN_PASSWORD="f58bae86cxxd59f891deb1b7b3c75a7> 12d652af8c59"
>
> VMS USAGE & QUOTAS
>
> VMS USAGE & QUOTAS - RUNNING
>
> DATASTORE USAGE & QUOTAS
>
> NETWORK USAGE & QUOTAS
>
> IMAGE USAGE & QUOTAS

# If you get an error message then the OpenNebula Daemon could not be started properly:
```

접속해보자.

http://<frontend_address>:9869

http://10.1.5.85:9869

![](./images/2021-08-25-18-51-00.png)

ui로 접속이 가능하다.

## for node

```sh
sudo -i

wget -q -O- https://downloads.opennebula.io/repo/repo.key | apt-key add -

echo "deb https://downloads.opennebula.io/repo/6.0/Ubuntu/20.04 stable opennebula" > /etc/apt/sources.list.d/opennebula.list

exit

sudo apt-get update -y
sudo apt-get install opennebula-node

sed -i -E 's/unix_sock_group.*/unix_sock_group\ \=\ \"oneadmin\"/gi' /etc/libvirt/libvirtd.conf
sed -i -E 's/unix_sock_rw_perms.*/unix_sock_rw_perms\ \=\ \"0777\"/gi' /etc/libvirt/libvirtd.conf
apt-get install -y opennebula-node-lxd
echo "setting oneadmin password..."
sudo passwd oneadmin
chown -R oneadmin:oneadmin /var/lib/libvirt/maas-images/
cd /var/lib/one
ssh oneadmin@kvm31 'cd /var/lib/one ; tar -czvf - .ssh' | tar -xzvf - -C .
```
