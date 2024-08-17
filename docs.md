# Extended Documentation

Welcome to the extended documentation for `relay.tools`!

- [Creating Additional Relays](#creating-additional-relays)
- [Logging in to Live Machines](#logging-in-to-live-machines)
- [Viewing the Logs](#viewing-the-logs)
- [Git Tracking](#git-tracking)
- [Disable Relay Creation](#disable-relay-creation)

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

>**Replace `YOUR.DOMAIN` to your own domain and `RELAY.YOUR.DOMAIN` to your relay's subdomain:**
>
>**In this example, we'll add `RELAY2.YOUR.DOMAIN` and `RELAY3.YOUR.DOMAIN`.**
```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "YOUR.DOMAIN" -d "RELAY.YOUR.DOMAIN" -d "RELAY2.YOUR.DOMAIN" -d "RELAY3.YOUR.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
```

>**Change both instances of `YOUR.DOMAIN` to your own domain:**
```
cat /srv/haproxy/certs/live/YOUR.DOMAIN/fullchain.pem /srv/haproxy/certs/live/YOUR.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem
```

- Now type ```exit``` and ```reboot```. Your relays should be certified! :)

# Git Tracking

`Relay.tools` contains a script to keep itself up to date.

You may wish to fork the `relaycreator` repo and maintain your own branch to update at your leisure.

- Replace `branch_name` and `YOUR.USERNAME` with your own details.
```
cd /srv/relaycreator/
git remote add -f [branch_name] https://github.com/YOUR.USERNAME/relaycreator
git checkout -b new [branch_name]/main
```

# Logging in to Live Machines

`Relay.tools` is comprised of many machines! Some include *haproxy, strfry, relaycreator,* and *keys-certs-manager*.

You can sign into them with `machinectl login` to make live changes in production.

For example, `machinectl login strfry`.

user: `root`

pass: `creator`

>**NOTE: Press `Ctrl + ]]]` to escape the login screen. You'll want to remember this!**

# Viewing the Logs

>In this example, we'll view the `strfry` logs.

- `login strfry`
- enter `user` and `pass`
- `journalctl -u interceptor.service -f`

>**Drop the `-f` to view all logs.**

You can also view the logs of an individual relay!

The `relay id` is available in the `url` when visiting the landing page of any relay you have created.

>It should look like `clzqm4zfl005doysgxppib0za`
```
login [relay_id]
journalctl -u interceptor.service -f
```

# Disable Relay Creation

You may wish to disable relay creation, especially if you do not have Lightning set up yet.
```
cd /srv/relaycreator/
sudo nano .env
```
- **Change `PAYMENTS_ENABLED=false` to `true`**
- `machinectl restart relaycreator`
