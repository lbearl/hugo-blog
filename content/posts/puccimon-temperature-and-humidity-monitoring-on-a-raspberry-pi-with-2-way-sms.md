---
title: "PucciMon - Temperature and Humidity monitoring on a Raspberry Pi with 2-way SMS"
date: "2018-01-31"
categories: 
  - "c"
  - "iot"
  - "python"
  - "software-development"
coverImage: "test-run.png"
---

Over the last weekend I decided it would be fun to brush off one of my old Raspberry Pis and play with some of the hardware that I bought for it a while back for a project I was going to do and subsequently abandoned. Overall what I wanted to accomplish was pretty simple: have a little C-based application which can read the AM2302 sensor which was attached to the Pi and use the Twilio REST interface via C in order to send messages.

I ended up going a little beyond my original goals since I discovered how easy Twilio makes it to respond to inbound SMS. I had a partially developed Flask application ([here](https://www.puccithe.dog)) which I could easily extend to have an additional method which Twilio could invoke.

Before going any further, the Python/Flask code is [here](https://github.com/lbearl/puccithe.dog) and the C code for the Pi is [here](https://github.com/lbearl/PucciMon). I haven't written a single line of C code in about 4 years, so I'm sure it's stylistically not the greatest code out there, but it is functional for my purposes.

## Wiring up the Pi

The first step in this project is to wire up the [AM2302 sensor](https://www.amazon.com/gp/product/B018JO5BRK) to the Raspberry Pi. Note for this project I'm using the Model B Pi 1 (I purchased it way back in 2013, and it's been repurposed a few times now). The AM2302 can be directly connected to the GPIO pins on the Pi. For wiring the positive lead (the black wire on the sensor I have), should go to pin 1. The negative lead (the grey wire) goes to pin 6. The white one goes to pin 7 (GPIO4 on the Pi).

The AM2302 should look similar to this: ![AM2302 picture](images/am2302.jpg)

After wiring up the RPi, it should look like this (ignore the case I have on mine): ![Wired Raspberry Pi](images/rpi-wiring.jpg)

Now that everything is wired up, the hardware portion of this project should be done.

## Twilio Account Setup

Before we go any further, you'll need a Twilio account in order to proceed. The next step will require us to know the From number, Twilio Account SID and Auth Token. If you head over to [Twilio](https://www.twilio.com/try-twilio), the onboarding experience is pretty painless.

## Development Environment Setup

If you haven't already booted up your Pi, do so now and make sure you have a good network connection. I'm assuming you've prepared the Pi with the default Raspbian Debian-derived Linux installation that most people use.

In order to make full use of everything, you'll need to get two sets of code: the C based application which handles querying the AM2302 and a second Python/Flask based application which handles the Twilio webhooks.

1. SSH to your Pi
2. `cd` to wherever you want the code to live
3. `git clone https://github.com/lbearl/PucciMon.git` to get the C code
4. `git clone https://github.com/lbearl/puccithe.dog.git` to get the python code
5. Set the following environment variables: a. `export TWIL_WEATHER_NUMBER=<Your From Number in E.164 format>` b. `export TWIL_WEATHER_AUTH=<Your Twilio Auth key>` c. `export TWIL_WEATHER_SID=<Your Twilio SID>` d. `export TWIL_WEATHER_TONUMS=<Comma separated list of recipients in E.164 format>`
6. `cd` into PucciMon and run `make` (of course you have to have all of the build tools installed)
7. `sudo mkdir -p /opt/lbearl/` in order to make directory for the file.
8. Execute `sudo -E ./bin/c_sms -f` and make sure that the AM2302 was read and that you receive an SMS. a. ![Test run output of application](images/test-run.png)
9. Execute `cat /opt/lbearl/temp.txt` and verify that it matches the output of the test run.
10. In order to make things slightly easier to execute, without always requiring sudo, I've found that setting UID works well:

```
sudo mkdir -p /opt/lbearl/bin
sudo cp ./bin/c_sms /opt/lbearl/bin/c_sms
chown root /opt/lbearl/bin/c_sms
chmod u+s /opt/lbearl/bin/c_sms
```

At this point the sensor is working properly and can be read, the remainder of the setup is getting the flask application up and running. I've opted to do this using WSGI on Apache. The relevant configuration is as follows:

```
<VirtualHost *:443>
        ServerName puccithe.dog
        ServerAlias www.puccithe.dog

        ServerAdmin luke@bearl.me

        WSGIDaemonProcess puccidog user=www-data group=www-data threads=5
        WSGIScriptAlias / /var/www/puccithe.dog/puccithedog.wsgi

        <Directory /var/www/puccithe.dog>
                WSGIProcessGroup puccidog
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # I use CloudFlare's certificates so that I don't have to muck about with anything else
        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/puccithedog-cf.pem
        SSLCertificateKeyFile /etc/ssl/private/puccithedog-cf.key
</VirtualHost>
```

The WSGI file is:

```
import sys
activate_this = '/home/pi/puccidog/puccidog/env/Scripts/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
sys.path.insert(0, '/home/pi/puccidog/puccidog')

import logging, sys
logging.basicConfig(stream=sys.stderr)

from puccidog import app as application
```

It's been a couple of years since I set that up (the Flask application was actually originally used for a different side project, and I just repurposed it to process the webhooks). If you look around on the [Flask website](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/), they have some good documentation around how to make this work.

## CloudFlare Dynamic DNS

As mentioned earlier, I'm using CloudFlare to handle all of the DNS and SSL around this. CloudFlare has a little known feature where they actually are able to use recent versions of `ddclient` in order to update your `A` records. I used to use Namecheap's Dynamic DNS, and the CloudFlare one is just as simple to set up. The one big gotcha is that the version of ddclient that ships with Raspbian by default isn't new enough to support CloudFlare. In order to fix it, we have to install a brand new version. Jens Segers wrote up a great summary on [how to do it](https://jenssegers.com/84/dynamic-dns-for-cloudflare-with-ddclient). The short version is that we need to download `ddclient` from SourceForge and then overwrite our local copy. The configuration file layout changed, so we need to create an `/etc/ddclient` directory and move (or create a new) `/etc/ddclient/ddclient.conf` in there. The configuration should look something like:

```
################
# ddclient.conf
################
use=web, web=dyndns
ssl=yes
protocol=cloudflare,
server=www.cloudflare.com,
zone=puccithe.dog,
login=<CloudFlare Email Address>
password=<CloudFlare API Key>
puccithe.dog,
```

## Testing

Now if you send an SMS to your Twilio "From" number, you should get a response back a few seconds later which has the most recent temperature data. Additionally, if you scheduled everything in `cron`, then any time a measurement comes in above 86F/30C, you'll get an SMS. You can also pass `-f` to the `c_sms` binary any time in order to force it to send SMS messages.

## Why?

Why did I build this? Mostly because it was fun, and also partly because sometimes I forget to turn on the AC and don't want my dog to have to suffer. I hope you enjoyed reading this as much as I enjoyed building it.
