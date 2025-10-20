<p align="center">
  <img src="rt.png" />
</p>
<br>

<p align="center">
  Welcome to the relay.tools installation guide!
</p>
<br>

If you already have an instance of `relay.tools` set up, you may be looking for the [documentation](docs.md)!

# Installation
```
git clone https://github.com/relaytools/relay-tools-images.git
cd relay-tools-images/machines
./prereqs.sh
./build
```
<p align="center">
  This is a good time to grab some coffee. ☕ (Average time: 15-20m)
</p>

# Configuration

## Pre-requisites.
- You must have configured a domain to point to this IP address, preferrably with a wildcard DNS address.
- Eg *.example.com -> your IP

>**Change `your.domain` to your own domain.**
```
export MYDOMAIN=your.domain
./configure.sh
machinectl enable mysql && machinectl enable strfry && machinectl enable relaycreator && machinectl enable haproxy
reboot
```

# Relay Creation

- Navigate to your domain in a browser and select the drop-down menu
- Sign in with Nostr (Authorize with NIP-07 extension)
- Create some relays!
- Feel free to check out some of the other [documentation](docs.md)!

# Other docs
Spiral's guide to `relay.tools` locally, [Local Install Guide](relaytools-setup.md)!
