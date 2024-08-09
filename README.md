```
▄▄▄  ▄▄▄ .▄▄▌   ▄▄▄·  ▄· ▄▌  ▄▄▄▄▄            ▄▄▌  .▄▄ · 
▀▄ █·▀▄.▀·██•  ▐█ ▀█ ▐█▪██▌  •██   ▄█▀▄  ▄█▀▄ ██•  ▐█ ▀. 
▐▀▀▄ ▐▀▀▪▄██ ▪ ▄█▀▀█ ▐█▌▐█▪   ▐█.▪▐█▌.▐▌▐█▌.▐▌██ ▪ ▄▀▀▀█▄
▐█•█▌▐█▄▄▌▐█▌ ▄▐█▪ ▐▌ ▐█▀·.   ▐█▌·▐█▌.▐▌▐█▌.▐▌▐█▌ ▄▐█▄▪▐█
.▀  ▀ ▀▀▀ .▀▀▀  ▀  ▀   ▀ • ▀  ▀▀▀  ▀█▄▀▪ ▀█▄▀▪.▀▀▀  ▀▀▀▀ 
```
# Detailed Installation Guides
 * [Spiral's Local Install Guide](relaytools-setup.md)
 * [Tekkadan's VPS Install Guide](relaytools-vps.md)

# Installation
```
git clone https://github.com/relaytools/relay-tools-images.git
cd /root/relay-tools-images/machines
./prereqs.sh
./build
```
<p align="center">
  This is a good time to grab some coffee. ☕ (Average time: 15-20m)
</p>

# Configuration

>**Change `your.domain` to your own domain:**
```
export MYDOMAIN=your.domain
./configure.sh
machinectl enable mysql && machinectl enable strfry && machinectl enable relaycreator && machinectl enable haproxy
reboot
```

# Relay Creation

- Navigate to your domain in a browser
- Sign in with Nostr (Authorize with NIP-07 extension)
- Create a relay from the dropdown menu

# Certificates

```
cd /root/relay-tools-images/machines
machinectl terminate haproxy
systemd-nspawn -M keys-certs-manager /bin/bash
```

>**Change `YOUR.DOMAIN` to your own domain and `RELAY.YOUR.DOMAIN` to your relay's subdomain:**
>
>**These are both case-insensitive.**
```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "YOUR.DOMAIN" -d "RELAY.YOUR.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
```
>**Change both instances of `YOUR.DOMAIN` (case-insensitive) to your own domain:**
>
>**These are both case-insensitive.**
```
cat /srv/haproxy/certs/live/YOUR.DOMAIN/fullchain.pem /srv/haproxy/certs/live/YOUR.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem
```

- Now type ```exit``` and ```reboot``` to complete your installation of relay.tools :)
