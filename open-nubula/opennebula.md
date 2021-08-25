# opennebula

<https://docs.opennebula.io/6.0/installation_and_configuration/frontend_installation/index.html>

## install db

```sh
sudo apt update -y
sudo apt -y install mariadb-server
sudo mysql_secure_installation
mysql -u root -p
```

## install docker

꼭 필요한가?

## install opennebula

```sh

sudo apt update -y
sudo apt  install gnupg wget apt-transport-https -y

wget -q -O- https://downloads.opennebula.io/repo/repo.key | apt-key add -

echo "deb https://downloads.opennebula.io/repo/6.0/Ubuntu/20.04 stable opennebula" > /etc/apt/sources.list.d/opennebula.list

sudo apt update -y

sudo apt install opennebula opennebula-sunstone opennebula-fireedge opennebula-gate opennebula-flow opennebula-provision -y

#usermod -a -G docker oneadmin

#sudo /usr/share/one/install_gems

sudo vim /etc/one/oned.conf
mysql -u oneadmin -p
sudo su - oneadmin
sudo ufw allow proto tcp from any to any port 9869

sudo systemctl start opennebula opennebula-sunstone

sudo systemctl enable opennebula opennebula-sunstone

sudo systemctl status opennebula

sudo systemctl status opennebula-sunstone

cd /var/lib/one/
ls -alF
su - oneadmin -c "oneuser show"
sudo systemctl status opennebula
sudo systemctl status opennebula-sunstone
lsof -i :9869
ip addr
virsh list

## haproxy
apt-get install -y haproxy

cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.orig
vi /etc/haproxy/haproxy.cfg

vi /etc/haproxy/haproxy.cfg
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy

vi /lib/systemd/system/haproxy.service
vi /etc/haproxy/haproxy.cfg
systemctl restart haproxy
systemctl status haproxy
sync && reboot
virsh list

systemctl status opennebula
systemctl status opennebula-sunstone

cat /etc/haproxy/haproxy.cfg

```
