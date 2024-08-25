/* Deprecated documentation being temporarily archived in the event that it becomes useful for later guide-building. */

# Manual Relay Creation
- Create a relay (you can create more later)
>**You will need to re-certify each time you create additional relays.**
>
>**See the [documentation](docs.md) if you would like to create more than one relay at this time.**

**In this example, we'll create `RELAY.YOUR.DOMAIN`.**

# Certificates

```
cd /root/relay-tools-images/machines
machinectl terminate haproxy
systemd-nspawn -M keys-certs-manager /bin/bash
```

>**Change `YOUR.DOMAIN` to your own domain and `RELAY.YOUR.DOMAIN` to your relay's subdomain.**
>
>**These are both case-insensitive.**
```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "YOUR.DOMAIN" -d "RELAY.YOUR.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
```
>**Change both instances of `YOUR.DOMAIN` to your own domain.**
>
>**These are both case-insensitive.**
```
cat /srv/haproxy/certs/live/YOUR.DOMAIN/fullchain.pem /srv/haproxy/certs/live/YOUR.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem
```

- Now type ```exit``` and ```reboot``` to complete your installation of relay.tools :)
