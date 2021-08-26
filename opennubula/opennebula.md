# opennebula

<https://docs.opennebula.io/6.0/installation_and_configuration/frontend_installation/index.html>

## install db mysql

```sh
sudo apt update -y
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

이제 접속해보자.

```sh
sudo mysql
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

## for node

```sh
wget -q -O- https://downloads.opennebula.org/repo/repo.key | sudo apt-key add -
sudo apt-get update
echo "deb https://downloads.opennebula.org/repo/5.12/Ubuntu/20.04 stable opennebula" | sudo tee /etc/apt/sources.list.d/opennebula.list
sudo apt-get update
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
