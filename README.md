# Detailed Installation Guides

 * [Spiral's Local Install Guide](relaytools-setup.md)

 * [Tekkadan's VPS Install Guide](relaytools-vps.md)

# Condensed Installation
```
apt install git && git clone https://github.com/relaytools/relay-tools-images.git
cd /root/relay-tools-images/machines
./prereqs.sh
./build
```
This is a good time to grab some coffee. ☕ (Average time: 15-20m)
- **NOTE: Be sure to change `EXAMPLE.DOMAIN` to your own domain:**
```
export MYDOMAIN=EXAMPLE.DOMAIN
./configure.sh
machinectl enable mysql && machinectl enable strfry && machinectl enable relaycreator && machinectl enable haproxy
reboot
```
- Navigate to your domain in a browser
- Sign in via NIP-07
- Create a relay, in this example it is named 'relay'

```
cd /root/relay-tools-images/machines
machinectl terminate haproxy
systemd-nspawn -M keys-certs-manager /bin/bash
```

- **NOTE: Be sure to change _EACH `EXAMPLE.DOMAIN`_ to your own domain:**

```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "EXAMPLE.DOMAIN" -d "RELAY.EXAMPLE.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive

cat /srv/haproxy/certs/live/EXAMPLE.DOMAIN/fullchain.pem /srv/haproxy/certs/live/EXAMPLE.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem

exit

reboot
```


