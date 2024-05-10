# relaytools
*2024-04-25 (1st draft) still needs some clean up

### This guide will help beginners setup a Nostr relay vending machine. 

**Why would I want this?**
If you want to on-the-fly be able to create custom Nostr relays for yourself, friends, or strangers for free or for sats! 

If this is the case, let's get started with relaytools!

**Prerequisites:** 
 - Linux (ubuntu server)
 - Wildcard DNS (noip.com)
 - Ports 80 and 443 opened for your server

**Methodology:**
 - Linux
	 - systemd/nspawn
		 - machine:mysql
		 - machine:relaycreator
		 - machine:strfry
		 - machine:haproxy
 - web interface on port 443

##  Linux server or Virtual Machine (ubuntu server)
*Skip this section if you already have a linux on the cloud. 

Jump to [Let's now set up relaytools](https://gist.github.com/SpiralCrunch/ec24f1f460bfd0b0870564fa07d0aaea#lets-now-set-up-relaytools).

We needs a Linux system to install relaytools, we will use Ubuntu server within this setup guide.
>This could be an old desktop or laptop you no longer use and want to repurpose, or one of the little multi-core sandwich sized computers.

I will use a VirtualBox VM with a single CPU and 4GBs of RAM

**Let's make the VM first:**
- Make a new VM within VirutalBox called "relaytools" or what ever you like.
 - We will be using ubuntu-22.04.4-live-server-amd64.iso as this is current at time or writing with 4096MBs of virtual RAM
- The dynamically sized disk will be 1000GBs (should this be more?)
- Set the Network to "Bridged Adapter" This will allow us an IP served from our LAN router
  - Goto Storage and attach the Ubuntu ISO to the virtual Optical Drive
  - Start the VM now and install the base OS (~10 minutes)
    - Let's choose:
      - "Ubuntu Server"
      - DHCP showed a local IP in the proper range for me 192.168.2.70
      - I removed the "x" from set up as LVM group
      - setup your name, server name, username and password
      - Put an "x" in to enable Install OpenSSH server
      - continue with the install...
      (~10 minutes)
      - Reboot and remove virtual optical disk
        
**Boot into the VM:**
  Login and get the IP to allow access via SSH

	ip addr |grep 192.168
    
    # You should get output like this
    inet 192.168.2.70/24 brd 192.168.2.255 scope global dynamic noprefixroute enp5s0
    
It shows my LAN IP is 192.168.2.70
- We will need this when SSH'ing into the server to perform the setup.
- Also this is needed for port forwarding setup later on.

## -   Wildcard DNS ([noip.com](http://noip.com/))
This was the only cost for me. I have used noip.com's free DNS for years but needed to pay for at least one wildcard service to use relaytools. I purchased "Enhanced Dynamic Dns" for 1 year with coupon code: *REFER20* for $15.99 USD*

This allows me to have ***.nostr-hub.ddns.net** meaning I have many, many subdomains such as follows:

 - wss://jacks-relay.nostr-hub.ddns.net
 - wss://jills-relay.nostr-hub.ddns.net
 - wss://bobs-relay.nostr-hub.ddns.net
 - wss://and so on...
 
****Not bad for $1.33/month***

Our goal is to setup `*.nostr-hub.ddns.net` within noip.com's control panel
(pending)

**Now we have to get your DNS/domain name setup**
(pending)

## SSH into your server and do the following
You will need your `username` and `ip address` to SSH into your server. Here is my example.

    ssh spiral@192.168.2.70
Enter your password and login

**As a normal user**

    cd ~
    wget https://www.noip.com/download/linux/latest
    tar xf latest
    cd ~/noip-duc_3.1.0/binaries
    ls -altr

* Note: The steps on the site https://my.noip.com/dynamic-dns/duc  are slightly wrong. (we should let them know)

**Install noip-duc (we will need to type our password for sudo)**
  
      sudo apt install ./noip-duc_3.1.0_amd64.deb

Launch the noip-duc app to connect to noip and start the WAN IP discovery for noip.com

    cd ~
    noip-duc -g all.ddnskey.com --username dnskeyusername --password dnskeypassword

**Output should look similar to the following:**

    spiral@relaytoolsv2:~$ noip-duc -g all.ddnskey.com --username dnskeyusername --password dnskeypassword
    [2024-04-22T03:47:23Z INFO  noip_duc] Starting update loop
    [2024-04-22T03:47:23Z INFO  noip_duc::public_ip] Attempting to get IP with method Dns(No-IP Anycast DNS Tools)
    [2024-04-22T03:47:24Z INFO  noip_duc] got new ip; ip=146.70.112.84, last_ip=0.0.0.0
    [2024-04-22T03:47:24Z INFO  noip_duc] update succeeded; ip=146.70.112.84, changed=false
    [2024-04-22T03:47:24Z INFO  noip_duc] checking ip again in 5m
		
This is now reporting your WAN or web facing IP address to [noip.com](http://noip.com/) so they can direct DNS queries to your server.

## Let's now set up relaytools
**Switch to root**

    sudo -i

**Now clone relaytools from the github repo** 

    cd /root
    git clone https://github.com/relaytools/relay-tools-images.git
    cd /root/relay-tools-images/machines && ls -altr

**install the prereqs for systemd-nspawn**

    ./prereqs.sh

**builds all the images (this takes a while ~29 minutes)**

    ./build

(pending)*Now that the build is finished we must now deal with The domain name and port forwarding before we continue as these need to be setup to allow the "Let's Encrypt" cert to be issued and 
installed correctly.

## Setup port forwarding for ports 80 and 443 within our router and test via LAN and WAN
Every router has a section that allows you to do port forwarding for IPs that your router assigns to the devices attached. Be sure to allow for your server IP both port 80 and 443
****How to actually do this is outside of the scope of this instruction set***

**Test with python3**

    mkdir ~/testfolder
    cd ~/testfolder
    touch nothing-to-see-here.txt
    sudo python3 -m http.server 80

If that does not work try:

    sudo python3 -m http.server 443

**Test internally with the IP in a browser on the same network**

http://192.168.2.70 *be sure to use your servers IP

Expected output:

    Directory listing for /
    nothing-to-see-here.txt

The same output should be shown for the external IP
  http://146.70.112.84/
  http://146.70.112.84:443/
  http://nostr-hub.ddns.net:443/

If that is all working, let's proceed...

# Let's switch back to root to finish up the setup of relaytools
Switch to root

    sudo -i

Navigate back to the machines folder and list the tools

    cd /root/relay-tools-images/machines && ls -altr

**Set an environment variable for your DNS with the following**
**Note:** This must be setup and tested before proceeding

    export MYDOMAIN=nostr-hub.ddns.net

Now run configure.sh

    ./configure.sh

Expected output

    Saving debug log to /srv/haproxy/certs/letsencrypt.log
    Account registered.
    Requesting a certificate for nostr-hub.ddns.net
    
    Successfully received certificate.
    Certificate is saved at: /srv/haproxy/certs/live/nostr-hub.ddns.net/fullchain.pem
    Key is saved at:         /srv/haproxy/certs/live/nostr-hub.ddns.net/privkey.pem
    This certificate expires on 2024-07-22.
    These files will be updated when the certificate renews.
    Certbot has set up a scheduled task to automatically renew this certificate in the background.
        - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
        If you like Certbot, please consider supporting our work by:
         * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
         * Donating to EFF:                    https://eff.org/donate-le
        - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

https://nostr-hub.ddns.net/
https://relay.nostr-hub.ddns.net/
https://nostr-hub.ddns.net/curator?

After we have created a new subdomain relay via the web interface we need to manually create the Let's Encrypt certs needed.

    machinectl terminate haproxy
Login to keys-certs-manager machine, sort of, and let's make the certs

    systemd-nspawn -D /var/lib/machines/keys-certs-manager -U --machine keys-certs-manager

>yo!  yeah that one, it doesn't probably have the full system init.. so there isn't the classic login like the other ones, to get into the bash you just systemd-nspawn -M keys-certs-manager /bin/bash (or other command)


## How to enable the subdomain
    systemd-nspawn -M keys-certs-manager /bin/bash

**Note:** Don't forget to replace my domain names with your domain name.
```certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "nostr-hub.ddns.net" -d "relay.nostr-hub.ddns.net" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive```

```certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" -d "relay.nostr-hub.ddns.net" --expand --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive```

```certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" -d "relay.nostr-hub.ddns.net" --expand --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive```

**Output**
```
root@keys-certs-manager / certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" -d "nostr-hub.ddns.net" -d "relay.nostr-hub.ddns.net" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
      Saving debug log to /srv/haproxy/certs/letsencrypt.log
      Missing command line flag or config entry for this setting:
      You have an existing certificate that contains a portion of the domains you requested (ref: /srv/haproxy/certs/renewal/nostr-hub.ddns.net.conf)

  It contains these names: nostr-hub.ddns.net

  You requested these names for the new certificate: nostr-hub.ddns.net, relay.nostr-hub.ddns.net.

  Do you want to expand and replace this existing certificate with the new certificate?
```
  (You can set this with the --expand flag)
  Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile /srv/haproxy/certs/letsencrypt.log or re-run Certbot with -v for more details.
  root@keys-certs-manager / certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "relay.nostr-hub.ddns.net" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive
  Saving debug log to /srv/haproxy/certs/letsencrypt.log
  Requesting a certificate for relay.nostr-hub.ddns.net

  Successfully received certificate.
  Certificate is saved at: /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/fullchain.pem
  Key is saved at:         /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/privkey.pem
  This certificate expires on 2024-07-22.
  These files will be updated when the certificate renews.
  Certbot has set up a scheduled task to automatically renew this certificate in the background.

  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  If you like Certbot, please consider supporting our work by:
   * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   * Donating to EFF:                    https://eff.org/donate-le
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -


cat /srv/haproxy/certs/live/nostr-hub.ddns.net/fullchain.pem /srv/haproxy/certs/live/nostr-hub.ddns.net/privkey.pem > /srv/haproxy/certs/bundle.pem

cat /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/fullchain.pem /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/privkey.pem >> /srv/haproxy/certs/bundle.pem
chmod 0600 /srv/haproxy/certs/bundle.pem




# Create bundle.pem for main domain
cat /srv/haproxy/certs/live/nostr-hub.ddns.net/fullchain.pem /srv/haproxy/certs/live/nostr-hub.ddns.net/privkey.pem > /srv/haproxy/certs/bundle.pem
chmod 0600 /srv/haproxy/certs/bundle.pem

# Append to bundle.pem the subdomain(s)
```cat /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/fullchain.pem /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/privkey.pem >> /srv/haproxy/certs/bundle.pem```

```chmod 0600 /srv/haproxy/certs/bundle.pem```


## machinectl commands

    machinectl list-images
    machinectl list
    
    machinectl enable mysql
    machinectl enable strfry
    machinectl enable relaycreator
    machinectl enable haproxy


**Set the hostname of your server (optional)**

      sudo hostnamectl set-hostname strfry

## Manually Configure .env to enable and configure ammount of sats to sell relays for

    ssh spiral@192.168.2.70
    .env next?

# As root
sudo -i

    cd /srv/relaycreator/ && ls -altr
    vim .env



## Monitor relaytools logs

    machinectl login strfry

Username/Password: `root/creator`

    journalctl -fn

## Remove or note?
This was filed in the repo

    Seems this was missing ...
    git clone https://github.com/relaytools/spamblaster.git /spamblaster
    cd /spamblaster
    go build -x
    cp spamblaster /usr/local/bin

***Thanks to cloud fodder the creator behind relay.tools**

> Written with [StackEdit](https://stackedit.io/).