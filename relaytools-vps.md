# VPS Setup Instructions

What you will need to begin:
- Fresh install Ubuntu on VPS
- Configure your SSH keys

SSH into your server and update it to the latest version:
```
sudo apt update
sudo apt upgrade
sudo do-release-upgrade
```

>If you wish to modify the welcome message and the name of the server, you just need to edit these two files: “/etc/motd” for the welcome message and “/etc/hostname” for the server VPS name.

Install Git and clone the repository:
```
sudo apt install git && git clone https://github.com/relaytools/relay-tools-images.git
```
Navigate into the 'machines' directory :
```
cd /root/relay-tools-images/machines && ls -altr 
```
Configure requirements:
```
./prereqs.sh 
```
Build it:
```
./build
```
>This is a good time to grab some coffee.

**NOTE: Be sure to change `example.domain` to your own domain!**
>Type these separately.

```
export MYDOMAIN=example.domain
```

```
./configure.sh
```
```
machinectl enable mysql && machinectl enable strfry && machinectl enable relaycreator && machinectl enable haproxy
```
Reboot the machine with `reboot`
>**(Optional)**: You can reference the steps at the bottom to create a restore point with timeshift.


TEST YOUR DOMAIN TO ENSURE IT WORKS

machinectl terminate haproxy

systemd-nspawn -M keys-certs-manager /bin/bash

certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" -d "DOMAIN_NAME_GOES_HERE" --expand --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive

exit

machinectl enable haproxy

reboot

timeshift --create "completed configuration of relay.tools" --skip-grub

TO ADD A RELAY:

machinectl terminate haproxy

systemd-nspawn -M keys-certs-manager /bin/bash

certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" -d "RELAY_NAME_GOES_HERE" --expand --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive

cat /srv/haproxy/certs/live/"RELAY_NAME_GOES_HERE"/fullchain.pem /srv/haproxy/certs/live/"RELAY_NAME_GOES_HERE"/privkey.pem >> /srv/haproxy/certs/bundle.pem

chmod 0600 /srv/haproxy/certs/bundle.pem

reboot


# Timeshift Instructions

If you'd like to use Timeshift to create rollback points:
```
sudo apt install timeshift
```
>**NOTE:** Before proceeding, use `df` to identify your drives. 

You should see a list:
```
root@host:~# df
Filesystem     1K-blocks     Used Available Use% Mounted on
udev             3005016        0   3005016   0% /dev
tmpfs             607172    15816    591356   3% /run
/dev/sda3      616083744 15364560 569350432   3% /
tmpfs            3035856        0   3035856   0% /dev/shm
tmpfs               5120        0      5120   0% /run/lock
tmpfs            3035856        0   3035856   0% /sys/fs/cgroup
/dev/sda2        2023272   220852   1681284  12% /boot
tmpfs             607168        0    607168   0% /run/user/0
```
Identify your main drive in the above list, in my case, /dev/sda3, then assign it:
``` 
timeshift --snapshot-device /dev/sda3 # CHANGE /DEV/SDA3 TO YOUR DESIRED BACKUP DRIVE
timeshift --create --comments "init"
```
```
timeshift --create "fresh build of relay.tools" --skip-grub
```
