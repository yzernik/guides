[ [Intro](README.md) ] -- [ [Preparations](raspibolt_10_preparations.md) ] -- [ [Raspberry Pi](raspibolt_20_pi.md) ] -- [ [Bitcoin](raspibolt_30_bitcoin.md) ] -- [ [Lightning](raspibolt_40_lnd.md) ] -- [ [Mainnet](raspibolt_50_mainnet.md) ] -- [ [**Bonus**](raspibolt_60_bonus.md) ] -- [ [FAQ](raspibolt_faq.md) ] -- [ [Updates](raspibolt_updates.md) ]

------

### Beginner’s Guide to ️⚡Lightning️⚡ on a Raspberry Pi

------

## Bonus guide: Email alerts

*Difficulty: intermediate*

If LND restarts, it would be nice to get an email alert to unlock the wallet. The key is to use `sendmail` in combination with Gmail. Setup is not as trivial as with a pure SMTP server, but these are hard to come by these days. So let's take it step by step.

I recommend to create a new Gmail address that is not linked to any of your other profiles. This guide will reference to it as `server-email@gmail.com` .

### Setup sendmail

* Open a "root" user session and install the necessary components  
  `$ sudo su -`  
  `$ apt-get install sendmail mailutils sendmail-bin`
* Edit the first line in `hosts` to include the following details (adjust for your own hostname)
  `$ nano /etc/hosts`

```
127.0.0.1       localhost localhost.localdomain RaspiBolt 
```

* Set up the authentication configuration with you own Gmail credentials 
  `$ mkdir -m 700 /etc/mail/authinfo/`  
  `$ cd /etc/mail/authinfo/`  
  `$ nano gmail-auth` 

```
AuthInfo: "U:root" "I:server-email@gmail.com" "P:your-password"
```

* Create the database map for sendmail   
  `$ makemap hash gmail-auth < gmail-auth`
* Configure sendmail to use Gmail server and credentials. Insert the following snipped **directly** above the first `MAILER_DEFINITIONS`  line, save & exit  
  `$ nano /etc/mail/sendmail.mc`

```
define(`SMART_HOST',`[smtp.gmail.com]')dnl
define(`RELAY_MAILER_ARGS', `TCP $h 587')dnl
define(`ESMTP_MAILER_ARGS', `TCP $h 587')dnl
define(`confAUTH_OPTIONS', `A p')dnl
TRUST_AUTH_MECH(`EXTERNAL DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
define(`confAUTH_MECHANISMS', `EXTERNAL GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
FEATURE(`authinfo',`hash -o /etc/mail/authinfo/gmail-auth.db')dnl
```

* Rebuild the configuration and restart sendmail  
  `$ make -C /etc/mail`  
  `$ /etc/init.d/sendmail reload`
* That's it. Let's try sending an email to your regular email address  
  `$ echo "This rocks!" | mail -s "RaspiBolt alert" your-regular-email@domain.com`

### Send email alert by script

Now we want to automate the way our node can send us a real email alert. For now, we mainly send an status update.

* Create a new shell script, save and exit  
  `$ cd /usr/local/bin/`
  `$ sudo nano systemd-sendmail.sh `

```bash
#!/bin/bash
# RaspiBolt email alert script
# /usr/local/bin/systemd-sendmail.sh
# make executable with: sudo chmod +x /usr/local/bin/systemd-sendmail.sh

MAILTO="your-regular-email@something.com"
MAILFROM="server-email@gmail.com"
#UNIT=$1
UNIT="lnd"

UNITSTATUS=$(systemctl status $UNIT)
TIMESTAMP=$(date "+%Y-%m-%d %T")

EXTRA=""
for e in "${@:2}"; do
  EXTRA+="$e"$'\n'
done

sendmail $MAILTO <<EOF
From:$MAILFROM
To:$MAILTO
Subject:Status mail for unit: $UNIT
Status report for unit: $UNIT
$TIMESTAMP
------------------------------------------------------------
$UNITSTATUS
------------------------------------------------------------
$EXTRA
EOF

echo -e "Status mail sent to: $MAILTO for unit: $UNIT"

```

* Make the script executable as a program  
  `$ sudo chmod +x systemd-sendmail.sh`
* Execute the script and check your inbox  
  `$ ./systemd-sendmail.sh`

### Create systemd unit for sendmail

The final step is to integrate the email script with our systemd unit files. For systemd to be able to call our script, we need to wrap it in its own systemd unit.

* Create new systemd unit `sendmail@.service`, save and exit  
  `$ sudo nano /etc/systemd/system/sendmail@.service`

```bash
# RaspiBolt LND Mainnet: systemd unit for sendmail script
# /etc/systemd/system/sendmail@.service

[Unit]
Description=Systemd Unit Mailer
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/systemd-sendmail.sh %i "Hostname: %H" "Machine ID: %m" "Boot ID: %b"
```

* ​



### Alerts from systemd

For me the most useful example is to send an email every time LND is started, so that I remember to unlock the wallet.

* Edit the systemd unit `lnd.service` by uncommenting the following line:  
  `$ sudo nano /etc/systemd/system/lnd.service`

```bash
OnFailure=systemd-sendmail@%n
```



* ​







---

<< Back: [Bonus guides](raspibolt_60_bonus.md) 