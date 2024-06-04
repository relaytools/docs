# VPS Setup Instructions

What you will need to begin:
- [Fresh install of Ubuntu on a VPS](https://www.digitalocean.com/community/tutorials/how-to-upgrade-to-ubuntu-22-04-jammy-jellyfish)
- [Configure your SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-22-04)


[Update Currently Installed Packages:](https://www.digitalocean.com/community/tutorials/how-to-upgrade-to-ubuntu-22-04-jammy-jellyfish#step-2-updating-currently-installed-packages)

- Before beginning the release upgrade, it’s safest to update to the latest versions of all packages _for the current release_.
- Begin by updating the package list:

>`sudo apt update`
 
- Next, upgrade installed packages to their latest available versions:

>`sudo apt upgrade`

>`sudo do-release-upgrade`

- Install Git and clone the repository:

>`sudo apt install git && git clone https://github.com/relaytools/relay-tools-images.git`

- Navigate into the 'machines' directory:

>`cd /root/relay-tools-images/machines && ls -altr`

- Configure requirements:

>`./prereqs.sh `

- Build it:

>`./build`

*This is a good time to grab some coffee.* ☕

**NOTE: Be sure to change `EXAMPLE.DOMAIN` to your own domain:**

>`export MYDOMAIN=EXAMPLE.DOMAIN`

>`./configure.sh`

>`machinectl enable mysql && machinectl enable strfry && machinectl enable relaycreator && machinectl enable haproxy`

>`reboot`

# Creating a relay

**Now, navigate to your domain in a browser.**

- Sign in:

<p align="center">
  <img src="https://github.com/TekkadanPlays/docs/assets/93434084/826bbd35-1e58-4cc1-ae01-a0f8e0d329ff">
</p>

- Create relay:

<p align="center">
  <img src="https://github.com/TekkadanPlays/docs/assets/93434084/75e993d9-b3ac-490f-82f3-fee785392e96">
</p>

- Let's name our example 'relay':

<p align="center">
  <img src="https://github.com/TekkadanPlays/docs/assets/93434084/ddd1906f-6757-429b-9757-a0b73299fe1c">
</p>

**Return to your terminal:**

>`cd /root/relay-tools-images/machines`

>`machinectl terminate haproxy`

>`systemd-nspawn -M keys-certs-manager /bin/bash`

**NOTE: Be sure to change EACH `EXAMPLE.DOMAIN` to your own domain:**

>`certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "EXAMPLE.DOMAIN" -d "RELAY.EXAMPLE.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive`

>`cat /srv/haproxy/certs/live/EXAMPLE.DOMAIN/fullchain.pem /srv/haproxy/certs/live/EXAMPLE.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem`

>`exit`

>`reboot`

Enjoy!
