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

>**NOTE: Be sure to change `EXAMPLE.DOMAIN` to your own domain:**
```
export MYDOMAIN=EXAMPLE.DOMAIN
./configure.sh
machinectl enable mysql && machinectl enable strfry && machinectl enable relaycreator && machinectl enable haproxy
reboot
```

# Relay Creation

- Navigate to your domain in a browser
- Sign in with Nostr
- Create a relay from the dropdown menu

# Certificates

```
cd /root/relay-tools-images/machines
machinectl terminate haproxy
systemd-nspawn -M keys-certs-manager /bin/bash
```

>**NOTE: Be sure to change EACH `EXAMPLE.DOMAIN` to your own domain:**

>**NOTE: Be sure to ALSO change `RELAY.EXAMPLE.DOMAIN` to your RELAY'S name:**

```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "EXAMPLE.DOMAIN" -d "RELAY.EXAMPLE.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive

cat /srv/haproxy/certs/live/EXAMPLE.DOMAIN/fullchain.pem /srv/haproxy/certs/live/EXAMPLE.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem

exit

reboot
```


