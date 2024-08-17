# Extended Documentation

Welcome to the extended documentation for `relay.tools`!

- [Additional Relay Creation](#additional-relay-creation)
- [Logging in to Live Machines](#logging-in-to-live-machines)
- [Viewing the Logs](#viewing-the-logs)
- [Git tracking](#git-tracking)
- [Disable Relay Creation](#disable-relay-creation)

# Additional Relay Creation

The installation guide should have provided you with one initial running relay.

>**We'll assume this to be `RELAY.YOUR.DOMAIN`**

You'll need to run through the certification process again to certify any new relays. 

Go ahead and create all of the relays you wish to create via your web browser, the same as in the installation guide.

>**We'll assume this to be `RELAY2.YOUR.DOMAIN` and `RELAY3.YOUR.DOMAIN` in this example.

We'll terminate `haproxy` and login to the `keys-certs-manager` machine:
```
cd /root/relay-tools-images/machines
machinectl terminate haproxy
systemd-nspawn -M keys-certs-manager /bin/bash
```
This time, we're going to include additional relays.

You will still need to include `YOUR.DOMAIN` and any existing relays, like `RELAY.YOUR.DOMAIN`.

**Append another `-d "RELAY.YOUR.DOMAIN"` for each relay, as necessary:**

```
certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "YOUR.DOMAIN" -d "RELAY.YOUR.DOMAIN" -d "RELAY2.YOUR.DOMAIN" -d "RELAY3.YOUR.DOMAIN" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
```
Here we have added `RELAY2.YOUR.DOMAIN` and `RELAY3.YOUR.DOMAIN`.

>**Change both instances of `YOUR.DOMAIN` to your own domain:**
```
cat /srv/haproxy/certs/live/YOUR.DOMAIN/fullchain.pem /srv/haproxy/certs/live/YOUR.DOMAIN/privkey.pem > /srv/haproxy/certs/bundle.pem
```

- Now type ```exit``` and ```reboot``` - your relays should now be certified. :)

# Logging in to Live Machines

This suite is comprised of many machines! 

Some of these you may want to access in production, such as *haproxy, strfry, mysql,* and *keys-certs-manager*!

You can log in to any of these with ```machinectl login```, for instance ```machinectl login strfry```.

- user: ```root```

- pass: ```creator```

>**Note**: Press ```Ctrl + ]]]``` to escape this login page. You'll want to remember this!

# Viewing the Logs

```
machinectl login strfry
```

- enter `user` and `pass`

```
journalctl -u interceptor.service -f
```

>Drop the `-f` if you wish to view all logs!

You can also do this to view your relays updating in real-time. To do this you'll need the `id` of the relay you wish to access.

Your `relay's id` will be in the `url` when visiting its landing page in a browser.

>**It will look something like `clzxx7nmv001igu5wwib7aoi1`**

```
machinectl login [relay_id]
journalctl -u interceptor.service -f
```

# Git Tracking

You may want to fork the `relay-tools-images` repo and maintain your own branch. 

Relay.tools includes a script for staying up-to-date. The following commands will create your own branch for your instance to follow, so you can manage updates at your leisure.

```
cd /srv/relaycreator/
git remote add -f [branch_name] https://github.com/YOUR_USERNAME/relaycreator
git checkout -b new [branch_name]/main
```

# Disable Relay Creation

You might want to disable relay creation if you don't have Lightning set up yet.

```
cd /srv/relaycreator/
nano .env
```

- **Change `PAYMENTS_ENABLED=false` to `true`**
- ```exit``` and ```machinectl restart relaycreator``` :)
