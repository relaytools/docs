/* Deprecated documentation being temporarily archived in the event that it becomes useful for later guide-building. */

- [Manual Relay Creation](#manual-relay-creation)
- [Certificates](#certificates)
- [Creating Additional Relays](#creating-additional-relays)

# Manual Relay Creation
- Create a relay (you can create more later)
>**You will need to re-certify each time you create additional relays.**
>
>**See the [documentation](docs.md) if you would like to create more than one relay at this time.**

**In this example, we'll create `RELAY.YOUR.DOMAIN`.**

# Creating Additional Relays

To create additional relays, we'll need to run through the certification process again.

>**You'll need to do this each time you wish to create additional relays.**

The installation guide should have left you with one functional relay.

We'll assume this is `RELAY.YOUR.DOMAIN`.
```
cd /root/relay-tools-images/machines
machinectl terminate haproxy
systemd-nspawn -M keys-certs-manager /bin/bash
```
In this example, we'll add `RELAY2.YOUR.DOMAIN` and `RELAY3.YOUR.DOMAIN`. 

We are just appending an additional `-d "RELAY.YOUR.DOMAIN"` for each relay you wish to certify.

You can add up to 99 relays at this time with certbot. If you choose to handle certifications in another way, you can host as many relays as you like.

>**Replace `YOUR.DOMAIN` with your own domain.**
>
>**Replace each instance of `RELAY.YOUR.DOMAIN` with your own relays' subdomains.**
```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "YOUR.DOMAIN" -d "RELAY.YOUR.DOMAIN" -d "RELAY2.YOUR.DOMAIN" -d "RELAY3.YOUR.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
```

>**Replace both instances of `YOUR.DOMAIN` with your own domain.**
```
cat /srv/haproxy/certs/live/YOUR.DOMAIN/fullchain.pem /srv/haproxy/certs/live/YOUR.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem
```

- Now type ```exit``` and ```reboot```. Your relays should be certified! :)
>**Remember to always include `YOUR.DOMAIN` and any existing relays when certifying new relays!**

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
