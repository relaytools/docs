# Extended Documentation

Welcome to the extended documentation for `relay.tools`!

- [Creating Additional Relays](#creating-additional-relays)
- [Git Tracking](#git-tracking)
- [Logging in to Live Machines](#logging-in-to-live-machines)
- [Viewing the Logs](#viewing-the-logs)
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
In this example, we'll add `RELAY2.YOUR.DOMAIN` and `RELAY3.YOUR.DOMAIN`. 

We are just appending an additional `-d "RELAY.YOUR.DOMAIN"` for each relay you wish to certify.

You can add up to 99 relays at this time.

>**Replace `YOUR.DOMAIN` to your own domain.**
>
>**Replace each instance of `RELAY.YOUR.DOMAIN` to your own relays' subdomains.**
```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "YOUR.DOMAIN" -d "RELAY.YOUR.DOMAIN" -d "RELAY2.YOUR.DOMAIN" -d "RELAY3.YOUR.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
```

>**Change both instances of `YOUR.DOMAIN` to your own domain.**
```
cat /srv/haproxy/certs/live/YOUR.DOMAIN/fullchain.pem /srv/haproxy/certs/live/YOUR.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem
```

- Now type ```exit``` and ```reboot```. Your relays should be certified! :)
>**Remember to always include `YOUR.DOMAIN` and any existing relays when certifying new relays!**

# Git Tracking

`Relay.tools` contains a script to keep itself up to date.

You may wish to fork the `relaycreator` repo and maintain your own branch to update at your leisure.

>**Replace `branch_name` and `YOUR.USERNAME` with your own details.**
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

>**`exit` will get you back to the login screen.**
>
>**NOTE: Press `Ctrl + ]]]` to escape the login screen. You'll want to remember this!**

# Viewing the Logs

In this example, we'll view the `strfry` logs.

- `machinectl login strfry`
- enter `user` and `pass`
- `journalctl -u interceptor.service -f`

>**You can drop the `-f` to view all logs.**

Cool, right?

You can also view the logs of an individual relay!

>**Be sure to `exit` the `strfry` machine if you are following along.**

The `relay id` is available in the `url` when visiting the landing page of any relay you have created.

>It should look like `clzqm4zfl005doysgxppib0za`
- `login [relay_id]`
- `journalctl -u interceptor.service -f`

Try publishing a note to your relay if no one is using them yet. You'll see the magic happen in real-time!

# Disable Relay Creation

You may wish to disable relay creation, especially if you do not have Lightning set up yet.
```
cd /srv/relaycreator/
sudo nano .env
```
- **Change `PAYMENTS_ENABLED=false` to `true`**
- `machinectl restart relaycreator`
