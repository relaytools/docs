# Extended Documentation

Welcome to the extended documentation for `relay.tools`!

- [Git Tracking](#git-tracking)
- [Logging in to Live Machines](#logging-in-to-live-machines)
- [Viewing the Logs](#viewing-the-logs)
- [Disable Relay Creation](#disable-relay-creation)

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

`Relay.tools` is comprised of 4 machines! *haproxy, strfry, relaycreator,* and *mysql*.

Machines are what systemd-nspawn calls containers.  They are separated from the host system and can be upgraded or changed using machine images.

You can change shell into them with `machinectl shell` to make live changes in production.

For example, `machinectl shell strfry`.

>**`exit or CTRL+D` will exit the shell back into the host machine.**
>
>**NOTE: Press `Ctrl + ]]]` to escape the login screen. You'll want to remember this!**

# Viewing the Logs

In this example, we'll view the `strfry` logs.

- `machinectl shell strfry`
- `journalctl -u interceptor.service -f`

>**You can drop the `-u interceptor.service` to view all logs for this container.**

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
- `machinectl terminate relaycreator`
- `machinectl start relaycreator`
