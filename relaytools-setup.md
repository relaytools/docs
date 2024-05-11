
# relaytools
*2024-04-25 (1st draft) still needs some clean up
*2024-05-10 (2nd draft) attempting more clean up

### This guide aims to help beginners setup a Nostr relay vending machine.

**Why would I want this?**
If you want to on-the-fly be able to create custom Nostr relays for yourself, friends, or strangers for free or for sats! 

If this is the case, let's get started with relaytools!

**Prerequisites:** 
 - Linux (ubuntu server)
 - Wildcard DNS (noip.com)
 - Ports 80 and 443 opened for your server

##  Linux server or Virtual Machine (ubuntu server)
*Skip this section if you already have a Linux in the cloud.  - Jump to [Let's now set up relaytools](https://gist.github.com/SpiralCrunch/ec24f1f460bfd0b0870564fa07d0aaea#lets-now-set-up-relaytools).

A Linux server system is needed to install relaytools, we will use Ubuntu server in this guide.

- We will use a VirtualBox VM with a single CPU and 4GB of RAM; however, this could be an old desktop or laptop you no longer use and want to repurpose, or one of the small, multi-core, sandwich-sized computers.

> Note: Installing VirtualBox is outside the scope of the guide. This
> overview might be helpful is you need it:
> https://www.nakivo.com/blog/use-virtualbox-quick-overview/

**Let's make the VM first:**
- Make a new VM within VirtualBox called "relaytools" or what ever you like.
-  We will be using [ubuntu-22.04.4-live-server-amd64.iso](https://releases.ubuntu.com/noble/ubuntu-24.04-live-server-amd64.iso)  as this is current at time of writing with 4096MBs of virtual RAM
-  The dynamically sized disk will be 1000GBs
- Set the Network to "Bridged Adapter" This will allow us an IP served from our LAN router
- Goto Storage and attach the Ubuntu ISO to the virtual Optical Drive
- Start the VM now and install the base OS (~10 minutes)
	- Let's choose:
	    - "Ubuntu Server"
	    - DHCP showed a local IP in the proper range; for myself I'll choose 192.168.2.70
	    - Let's removed the "x" from set up as LVM group
	    - Setup your name, server name, username and password
	    - Put an "x" in to enable Install OpenSSH server
	    - continue with the install...
      (~10 minutes)
      - Remove virtual optical disk and reboot
        
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

This allows for ***.nostr-hub.ddns.net**, meaning I have many, many subdomains, such as the following:

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
		
noip-duc is now reporting your WAN or web facing IP address to [noip.com](http://noip.com/) so they can direct DNS queries to your server.

**(pending) Need steps to setup this up as a service to autostart after reboot**

## Let's now set up relaytools
**Switch to root**

    sudo -i

**Now clone relaytools from the github repo** 

    cd /root
    git clone https://github.com/relaytools/relay-tools-images.git
    cd /root/relay-tools-images/machines && ls -altr

**install the prereqs for systemd-nspawn**

    ./prereqs.sh

**builds all the images (this takes a while ~15-30 minutes)**

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

If the domain and subdomains are working you will be able to test as follows:
http://spirals-archive.ddns.net/
http://what.spirals-archive.ddns.net/

https://image.nostr.build/bff870c4815c74ed1ce0ee7c968b94bffa983e3f50f8e3d57b84b78a8bce261f.png

https://image.nostr.build/04ea7620e7a3853798bd4beefa4d2b2f1a10962ad241ae32cda1fabc5682584f.png

Now test port 443 is accessible:

    sudo python3 -m http.server 443

https://image.nostr.build/3bcfb170e629eb8cf1fb1fdf8fdf0bdb32174fad77685cef4b0d20c0fefd5fbb.png

https://image.nostr.build/2d9d47674e230dd1a0a0c71bbae9b6c58ba9ff10713bab1f6b84d73afba54ece.png



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

**You should now have a working relaytools**
https://image.nostr.build/d6671bc42f234c2c2c0646366a39bd9749dc5c40084c90f7ef729707a9c79d78.png


Subdomain
https://image.nostr.build/ff4fc6f743f734f0c11a3703fa3d4e1c865d2762e51a5dbdd36e45fd90760ad2.png


https://nostr-hub.ddns.net/
https://relay.nostr-hub.ddns.net/
https://nostr-hub.ddns.net/curator?

## Manually create the Let's Encrypt certs
After we have created a new subdomain relay via the web interface we need to manually create the Let's Encrypt certs needed.

Switch to root

    sudo -i

    machinectl terminate haproxy

Login to keys-certs-manager machine, sort of, and let's make the certs

### How to enable the subdomain
Let's now access the bash interface for the keys-certs-manager machine

    systemd-nspawn -M keys-certs-manager /bin/bash

We need to create the cert now:
**Note:** Don't forget to replace my domain names with your domain name.


```certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" -d "tutorial.spirals-archive.ddns.net" --expand --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive```

**Output**
```
Saving debug log to /srv/haproxy/certs/letsencrypt.log
Requesting a certificate for tutorial.spirals-archive.ddns.net

Successfully received certificate.
Certificate is saved at: /srv/haproxy/certs/live/tutorial.spirals-archive.ddns.net/fullchain.pem
Key is saved at:         /srv/haproxy/certs/live/tutorial.spirals-archive.ddns.net/privkey.pem
This certificate expires on 2024-08-08.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**copy the pem files to the correct locations**
```
cd /srv/haproxy/certs/live/
ls -ltr
```
*This should show folders that have the same names as your domain and subdomains

Mine are as follows:
```
drwxr-xr-x 2 root root 4096 May 10 23:26 spirals-archive.ddns.net
drwxr-xr-x 2 root root 4096 May 10 23:35 tutorial.spirals-archive.ddns.net
```
spirals-archive.ddns.net
tutorial.spirals-archive.ddns.net

*I need to take these folder names and substitute within the following:

## Create bundle.pem for main domain
cat /srv/haproxy/certs/live/**spirals-archive.ddns.net**/fullchain.pem /srv/haproxy/certs/live/**spirals-archive.ddns.net**/privkey.pem > /srv/haproxy/certs/bundle.pem

chmod 0600 /srv/haproxy/certs/bundle.pem

```cat /srv/haproxy/certs/live/spirals-archive.ddns.net/fullchain.pem /srv/haproxy/certs/live/spirals-archive.ddns.net/privkey.pem > /srv/haproxy/certs/bundle.pem```

## Append to bundle.pem the subdomain(s)
```cat /srv/haproxy/certs/live/tutorial.spirals-archive.ddns.net/fullchain.pem /srv/haproxy/certs/live/tutorial.spirals-archive.ddns.net/privkey.pem >> /srv/haproxy/certs/bundle.pem```

```chmod 0600 /srv/haproxy/certs/bundle.pem```

Let's exit out of the haproxy container 

	exit
Expected output:

	Container keys-certs-manager exited successfully.

Now that we are back to the host Linux we need to enable all the container to auto start on reboot now:	
```
machinectl enable mysql
machinectl enable strfry
machinectl enable relaycreator
machinectl enable haproxy
```
Expected output:

```
Created symlink /etc/systemd/system/machines.target.wants/systemd-nspawn@mysql.service → /usr/lib/systemd/system/systemd-nspawn@.service.
Created symlink /etc/systemd/system/machines.target.wants/systemd-nspawn@strfry.service → /usr/lib/systemd/system/systemd-nspawn@.service.
Created symlink /etc/systemd/system/machines.target.wants/systemd-nspawn@relaycreator.service → /usr/lib/systemd/system/systemd-nspawn@.service.
Created symlink /etc/systemd/system/machines.target.wants/systemd-nspawn@haproxy.service → /usr/lib/systemd/system/systemd-nspawn@.service.
```
Now we reboot the host system

	reboot



https://image.nostr.build/c008a66424a27b0e43b245551db195a3c469c4e4adbc3b0efe61c75e58f0d850.png

https://image.nostr.build/dfb0ad37b791f73398d8b0029226faf52c2d688284e73b275bd7e20ee3e58218.png

# Congratulation! You now have a working relay made with relaytools!
**Thanks to cloud fodder the creator behind relay.tools**
> Written with [StackEdit](https://stackedit.io/).

## The following info likely needs removing...

machinectl terminate haproxy

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

**Methodology:**
 - Linux
	 - systemd/nspawn
		 - machine:mysql
		 - machine:relaycreator
		 - machine:strfry
		 - machine:haproxy
 - web interface on port 443

    systemd-nspawn -D /var/lib/machines/keys-certs-manager -U --machine keys-certs-manager

>yo!  yeah that one, it doesn't probably have the full system init.. so there isn't the classic login like the other ones, to get into the bash you just systemd-nspawn -M keys-certs-manager /bin/bash (or other command)



```certbot certonly --config-dir="/srv/haproxy/certs" --work-dir="/srv/haproxy/certs" --logs-dir="/srv/haproxy/certs" --expand -d "nostr-hub.ddns.net" -d "relay.nostr-hub.ddns.net" --agree-tos --register-unsafely-without-email --standalone --preferred-challenges http --non-interactive```



Navigate into that folder:
```
cd spirals-archive.ddns.net
ls -altr

cat /srv/haproxy/certs/live/nostr-hub.ddns.net/fullchain.pem /srv/haproxy/certs/live/nostr-hub.ddns.net/privkey.pem > /srv/haproxy/certs/bundle.pem
```
cat /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/fullchain.pem /srv/haproxy/certs/live/relay.nostr-hub.ddns.net/privkey.pem >> /srv/haproxy/certs/bundle.pem
chmod 0600 /srv/haproxy/certs/bundle.pem




**Thanks to cloud fodder the creator behind relay.tools**
> Written with [StackEdit](https://stackedit.io/).
