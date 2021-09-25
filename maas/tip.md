# tip

## cloud-init disable

maas로 설치를 하게 되면 cloud-init이 설치가 된다. 그래서 cloud-init를 안쓰고 싶은 경우에는 cloud-init을 disable해줘야한다.

* Prevent start

```bash
sudo touch /etc/cloud/cloud-init.disabled
```

* Disable all services \(uncheck everything except "None"\):

```bash
sudo dpkg-reconfigure cloud-init

# Uninstall the package and delete the folders

sudo dpkg-reconfigure cloud-init
sudo apt-get purge cloud-init
sudo rm -rf /etc/cloud/ && sudo rm -rf /var/lib/cloud/

# Restart the computer

sudo reboot
```

## 관련 폴더

* tftp 관련 폴더

/var/snap/maas/common/maas/boot-resources/current

* 로그 관련

/var/snap/maas/common/log/rsyslog/

