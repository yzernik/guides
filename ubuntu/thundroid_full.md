# The perfect Bitcoin Lightning️ node

![img](images/thundroid_banner.jpg)

## Introduction

Bitcoin is a native digital currency for the internet age. It can be thought of as just another international currency, but without a native country, so it defies borders, trade policies, local laws and arbitrary inflation. In the 2008 [whitepaper](https://bitcoin.org/bitcoin.pdf) by the pseudonymous Satoshi Nakamoto it is described as:

> A purely peer-to-peer version of electronic cash [which] would allow online payments to be sent directly from one party to another without going through a financial institution. 

Being peer-too-peer means that it does not rely on a middleman (eg. a bank) and can be transfered as a bearer asset, like physical cash, without asking anyone for permission. With the added benefit that it does not need to be stored physically, as it is secured by a cryptographic key, and can be transferred within minutes to anywhere in the world. One key component of this free open-source financial system is the blockchain, the ledger that keeps track of who owns how many bitcoin and that is stored as an identical copy by all users that decide to run a full Bitcoin node. (Learn more on [bitcoin.org](https://bitcoin.org))

Bitcoi is a economic experiment of epic scope, and it's success is by no means certain. In any case, *Bitcoin as a new technology* is an incredibly interesting endeavor, especially because of its interdisciplinary nature and the very low barrier to entry. *Bitcoin as sound money*, being scarce and non-inflationary, challenging money as one of the last true monopolies of nation states, could have a major impact on economic principles and society as a whole. 

At the moment, Bitcoin is more a *store of value* and not really suited for small everyday transactions. Truly **decentralized blockchains are a scarce resource** and cannot scale to accommodate all global transactions. If you think about it, it cannot be good practice to store every coffee purchase redundantly all over the world for all eternety. These limitations are a great motivator to build better technology to scale exponentially, as opposed to just making everything bigger for linear scaling.

This is where the **Lightning Network** comes in. As one of several new blockchain “extensions”, its promise is to accommodate nearly unlimited transactions, with instant confirmation, minimal fees and increased privacy. It sounds almost too good to be true, but this technology is well researched, committed to the cypherpunk open-source ethos and leverages the solid underpinnings of Bitcoin. (Learn more: [bit.ly/2vWO8Yf](https://bit.ly/2vWO8Yf))

To preserve the decentralized nature of this monetary system, I think **it is important that everybody can run their own trustless Bitcoin node**, preferably on cheap hardware like Odroid.

---

⚠️ *Please be aware that while Bitcoin has been battle-tested for almost a decade, the Lightning Network is still in beta and under heavy development. This guide allows you to set up your Bitcoin node, the Lightning part is optional. Read the "financial best practices part" before commiting real bitcoin to your node.*

---

### Purpose

My aim is to set up a **trustless Bitcoin Core and Lightning node** that runs 24/7, is part of and supports the decentralization of the Lightning network by routing payments and can be used to send and receive personal payments using the command line interface.

This server is set up **without graphical user** **interface** and is used remotely using the Secure Shell (SSH) command line. In the future, this server should function as my **personal backend for desktop and mobile wallets**, but I haven’t found a good solution to this yet. So, command line it is for the moment.

### Target audience

This guide strives to give simple and foolproof instructions. But the goal is also to do everything ourselves, no shortcuts that involve trust in a 3rd party allowed. This makes this guide quite technical, but I try to make it as straightforward as possible and explain everything for you to gain a basic understanding of the how and why.

Also, please be aware that all Lightning implementations are still in alpha phase and bugs are to be expected. If you like to learn about Linux, Bitcoin and Lightning, this guide is for you.

## 1) System setup

### Choosing the Odroid HC2

After publishing the "Beginner's Guide to Lightning on a Raspberry Pi", I started to explore other hardware, as the Pi has drawbacks, mainly in the area of performance and the the hassle with attaching external storage which is especially important when storing the whole Bitcoin blockchain. 

The [Hardkernel Odroid HC2](http://www.hardkernel.com/main/products/prdt_info.php) (or the HC1 for a smaller form factor) as a Linux-based mini PC is a perfect fit. Compared to a Rasperry Pi, it has the following advantages:

* price comparable to Raspberry Pi
* more powerful (8 core CPU, 2 GB RAM, Gigabit Ethernet)
* internal housing of harddisk, direct connection using SATA3
* only one power adapter for everything

Not available are features like HDMI output, built-in Wifi or GPIO pins, but these are not relevant to this project. Performance is way better, thus it seems more future-proof as Bitcoin and Lightning are certain to evolve.

![img](images/odroid_hc2_performance.png)*Performance of Odroid HC2 is identical to XU4 (which is more of a media pc)*

I decided to try this setup, ordered the Odroid HC2 and, after setting it up and running it for two month now, I think this is as good as it gets for a low-cost Bitcoin / Lightning node. As this project needs a cheesy name as well, I’ll call my node **Thundroid**. :-)

I ordered the following items directly with Hardkernel in Singapore. There are [resellers](http://www.hardkernel.com/main/distributor.php) available worldwide, but not for Switzerland, unfortunately.

* [Hardkernel Odroid HC2](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G151505170472) ($54)
* [Power adapter & cord](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G151578376044) ($7)
* optional: [Odroid HC2 case](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G151596669222) ($5)

You also need the following:

* Micro SD card: 8 GB or more, incl. adapter to your regular computer
* Internal hard disk: 500 GB or more, SATA, 3.5" hdd, or 2.5" hdd/ssd
* Network RJ45 cable

Assembly is easy, just enter the hard disk and fix it with the the screws included with your Odroid. If you ordered the plastic case, just slide it over the metal body. 

### Installing the operating system

We use Ubuntu 16.04.03 LTE (Minimal, Bare OS) that is provided by Hardkernel. Download the image from the Odroid-XU4 section on [wiki.odroid.com](https://wiki.odroid.com/).

Exact file used: [ubuntu-16.04.3-4.14-minimal-odroid-xu4-20171213.img.xz](https://odroid.in/ubuntu_16.04lts/ubuntu-16.04.3-4.14-minimal-odroid-xu4-20171213.img.xz)

Download the image, flash it on your MicroSD card, put it into your Thundroid, connect it to your network via cable and connect the power adapter. The initial boot can take several minutes. 

Configure your network router to assign a static IP address to your Thundroid. After rebooting the HC2, as SSH is on by default, you can connect using your favorite SSH client.

## 2) Working on your Thundroid

### Write down your passwords

You will need several passwords and I find it easiest to write them all down in the beginning, instead of bumping into them throughout the guide. They should be unique and very secure, at least 12 characters in length. Do not use uncommon special characters, blanks or quotes (‘ or “).

```
[ A ] User password
[ B ] Bitcoin RPC password
[ C ] Lightning API password
```

Store a copy of your passwords somewhere safe (preferably in a password manager like KeePass or LastPast) and keep your original notes out of sight once your system is up and running.

### The command prompt

Everything is configured on the Linux command prompt. Throughout this guide I use the following notation:

```
# this is a comment, just for information

$ command            This is a single-line command to enter (without the $) 
                     and confirm with the enter key

No prefix            This is either an output of the command above
                     or something you can copy/paste into a file
```



When you enter commands, you can use the “Tab” key for autocompletion, eg. for commands, directories or filenames.

If you are using Windows, I recommend to use the SSH client [KiTTY](http://kitty.9bis.com). You can **copy** text from the shell by selecting it with your mouse (no need to click anything), and **paste** stuff with a right-click.

### Basic configuration

Connect to the node using SSH. The access credentials are user: `root ` and password: `odroid`.

```
# change root password to [password A]
$ passwd 

# update the operating system
$ apt update
$ apt upgrade
$ apt dist-upgrade
$ apt install linux-image-xu3 
# answer [y], then [no] (do not abort)

# install some additional software
$ apt install htop git curl bash-completion jq

# set time zone & localization
$ dpkg-reconfigure tzdata
$ dpkg-reconfigure locales
```

When using the Nano text editor, you can use the same keyboard shortcuts  to save (Ctrl-O, confirm or change filename & press enter) and exit (Ctrl-X).

```
# change hostname (replace "odroid" with "thundroid" :)
$ nano /etc/hostname
$ nano /etc/hosts

# disable Swap file (this would degrade the MicroSD card)
$ swapoff --all

# create user "admin" and change password to [password A]
$ useradd -m admin
$ adduser admin sudo
$ passwd admin

# create user "bitcoin" and change password to [password C]
$ useradd -m bitcoin
$ passwd bitcoin
```

### Mounting the hard disk

```
# get NAME for hard disk
$ lsblk -o UUID,NAME,FSTYPE,SIZE,LABEL,MODEL

# format hard disk (use [NAME] from above, e.g /dev/sda1)
$ mkfs.ext4 /dev/[NAME]

# get UUID for hard disk, copy into notepad
$ lsblk -o UUID,NAME,FSTYPE,SIZE,LABEL,MODEL

# edit fstab and enter new line (replace UUID) at the end 
$ nano /etc/fstab
UUID=123456 /mnt/hdd ext4 noexec,defaults 0 0

# create mount point, mount, check and set owner
$ mkdir /mnt/hdd
$ mount -a
$ df /mnt/hdd
Filesystem     1K-blocks      Used Available Use% Mounted on
/dev/sda1      961300808 600388836 312057600  66% /mnt/hdd

$ chown bitcoin:bitcoin /mnt/hdd/

# make sure user "admin" uses bash as standard shell
$ sudo chsh admin -s /bin/bash
# restart
$ shutdown -r now
```

## 3) Hardening your Thundroid

Your Thundroid will handle money and needs to be super-secure. Login as “admin” (we will not use “root” again).

### fail2ban

Fail2ban monitors SSH login attempts and bans a remote peer after 5 unsuccessful tries for 10 minutes.

```
$ sudo apt install fail2ban
```

### UFW: Uncomplicated Firewall

The firewall denies all connection attempts from other peers by default and allows only specific ports to be used.

The line `ufw allow from 192.168.0.0/24 …` below assumes that the IP address of your Pi is something like `192.168.0.???`, the ??? being any number from 0 to 255. If your IP address is `12.34.56.78`, you must adapt this line to `ufw allow from 12.34.56.**0**/24 …`.

```
# change session to "root"
$ sudo su

$ apt install ufw
$ ufw default deny incoming
$ ufw default allow outgoing
$ ufw allow from 192.168.0.0/24 to any port 22 comment 'allow SSH from local LAN'
$ ufw allow 9735 comment 'allow Lightning'
$ ufw deny 8333 comment  'deny Bitcoin mainnet'
$ ufw allow 18333 comment 'allow Bitcoin testnet'
$ ufw enable
$ systemctl enable ufw
$ ufw status

# exit "root" session back to "admin"
$ exit
```

### SSH Keys

We will disable login via password completely and require a SSH certificate. To create it for your “admin” user, please follow this guide:

[**Configure "No Password SSH Keys Authentication" with PuTTY on Linux Servers***SSH ( Secure SHELL) is one of the most used network protocol to connect and login to remote Linux servers, due to its…*www.tecmint.com](https://www.tecmint.com/ssh-passwordless-login-with-putty/)

If you can log in as “admin” with your SSH key (check!), we disable the password login.

```
$ sudo nano /etc/ssh/sshd_config
# change ChallengeResponseAuthentication and PasswordAuthentication 
# to "no" (uncomment if necessary), save and exit
 
# copy the SSH public key for user "root", just in case
$ sudo mkdir /root/.ssh
$ sudo cp /home/admin/.ssh/authorized_keys /root/.ssh/
$ sudo chown -R root:root /root/.ssh/
$ sudo chmod -R 700 /root/.ssh/

$ sudo systemctl restart sshd.service
```

You can now only login with “admin” or “root” and your SSH key. As you cannot connect a screen to the Odroid, SSH is your only option.

⚠️ **Backup your SSH key! There is no fallback login!**

Worst case scenario: you need to flash the MicroSD card and set up the system again, all important stuff is still on the harddrive.

## Installing Bitcoin

Installing the Bitcoin Core software connects our node to the Bitcoin network, and allows us to use it also as a trustless Full Node.

```
$ mkdir /home/admin/download
$ cd /home/admin/download

# download the latest Bitcoin Core ARM binaries (check https://bitcoin.org/en/download)
$ wget https://bitcoin.org/bin/bitcoin-core-0.16.0/bitcoin-0.16.0-arm-linux-gnueabihf.tar.gz
$ wget https://bitcoin.org/bin/bitcoin-core-0.16.0/SHA256SUMS.asc
$ wget https://bitcoin.org/laanwj-releases.asc

# check that the reference checksum matches the real checksum
# output: "bitcoin-0.16.0-arm-linux-gnueabihf.tar.gz: OK"
$ sha256sum --check SHA256SUMS.asc --ignore-missing

# manually check the fingerprint of the public key:
# 01EA 5486 DE18 A882 D4C2  6845 90C8 019E 36C2 E964
$ gpg --with-fingerprint ./laanwj-releases.asc

# import the public key and verify the signed checksum file
# output: "Good signature from Wladimir ..."
# check the fingerprint again in case of malicious keys 
# 01EA 5486 DE18 A882 D4C2  6845 90C8 019E 36C2 E964
$ gpg --import ./laanwj-releases.asc
$ gpg --verify SHA256SUMS.asc

# extract, install and check active version
$ tar -xvf bitcoin-0.16.0-arm-linux-gnueabihf.tar.gz
$ sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-0.16.0/bin/*
$ bitcoind --version
Bitcoin Core Daemon version v0.16.0

# configuration
$ sudo su bitcoin
$ mkdir /mnt/hdd/bitcoin_testnet
$ ln -s /mnt/hdd/bitcoin_testnet /home/bitcoin/.bitcoin

# copy/paste the following Bitcoin configuration file 
# (replace PASSWORD_[E] with your password)
$ nano /home/bitcoin/.bitcoin/bitcoin.conf
```

Content of the file *bitcoin.conf*:

```
# Example bitcoin.conf for Thundroid
server=1
daemon=1
testnet=1
txindex=1

# Connection settings
#externalip=111.222.333.444   # to announce fixed ip
#port=12345                   # for non-standard port
rpcuser=bitcoin
rpcpassword=PASSWORD_[E]
zmqpubrawblock=tcp://127.0.0.1:29000
zmqpubrawtx=tcp://127.0.0.1:29000
```



```
# back to user "admin"
$ exit

# copy/paste the following systemd unit file
$ sudo nano /etc/systemd/system/bitcoind.service
```

Content of the file *bitcoind.service*:

```
[Unit]
Description=Bitcoin daemon
After=network.target

# for use with sendmail alert
#OnFailure=systemd-sendmail@%n

[Service]
User=bitcoin
Group=bitcoin
Type=forking
PIDFile=/home/bitcoin/.bitcoin/bitcoind.pid
ExecStart=/usr/local/bin/bitcoind -pid=/home/bitcoin/.bitcoin/bitcoind.pid
KillMode=process
Restart=always
TimeoutSec=120
RestartSec=30

[Install]
WantedBy=multi-user.target
```



```
# Enable the unit file and start it manually
$ sudo systemctl enable bitcoind
$ sudo systemctl start bitcoind
$ systemctl status bitcoind

# check bitcoind operations (exit with Ctrl-C)
$ sudo tail -f /home/bitcoin/.bitcoin/testnet3/debug.log

# check bitcoin blockchain verification progress
$ sudo su bitcoin
$ bitcoin-cli getblockchaininfo

# check public visibility
$ curl -sL https://bitnodes.earn.com/api/v1/nodes/me-18333/ | jq
{
  "success": true
}
```



## 4) Lightning Network















#### Considerations for Bitcoin mainnet

This guide is configured to use the Bitcoin testnet. You can easily download and verify the blockchain using the Odroid HC2. However, for a switch to Bitcoin mainnet, will will need to initially download and verify the Bitcoin blockchain on a regular computer (only once, not regularly).

There are multiple options (not covered in this guide) to transfer the mainnet blockchain (~200 GB) to your node:

* connect your internal hard disk directly to your computer,
* transfer the blockchain over the network
* use an additional external hard disk to transfer the data via USB.

We are going to format the internal hard disk using the Ext4 file system, which is not compatible with windows. If you want to connect the hard disk later to a windows computer, you need [additional software](http://www.paragon-drivers.com/extfs-windows/) (free for once-time-usage).

#### Assemble the hardware

Assembly is easy, just enter the hard disk and fix it with the the screws included with your Odroid. If you ordered the plastic case, just slide it over the metal body.

---

Now, apart from a few quirks, you can follow the [generic Ubuntu 16.04 guide](README.md).

---
