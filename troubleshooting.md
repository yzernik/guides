---
layout: default
title: Troubleshooting
nav_order: 20
---
# Troubleshooting
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

The aim of this additional troubleshooting guide is to help you debug your system, verify all important configurations and find the frikkin' little problem that keeps you from running the perfect Lightning node.

Where possible, I'll link to the relevant part of the guide. If you see any discrepancies from the expected output, double check how you set up your node in that specific area.

### [General FAQ](faq.md)

You can find frequent questions not directly related with issues in a separate [General FAQ](raspibolt_faq.md) section:

* Can I get rich by routing Lightning payments?
* Can I attach the Ext4 formatted hard disk to my Windows computer?
* What do all the Linux commands do?
* Where can I get more information?

### Hardware & operating system

#### Can you login using SSH?

Yes! The username is `umbrel`, the password is `moneyprintergobrrr`.

#### Do you have a compatible Raspberry Pi?

If you have a Raspberry Pi with at least 4GB of RAM, you can run Umbrel on it.

#### Is you root filesystem read-only?

If you get an error like `unable to ..... : Read-only file system`, this points to a faulty microSD card. If linux detects a corrupt root filesystem, it drops into read-only mode. Try to flash the microSD card again, or use a different card.

#### My Umbrel node keeps crashing. What can I do to fix the issue?
If you're not using the official power supply, it's probably the power supply.
To detect undervoltage, connect to your node via SSH and run this command: `vcgencmd get_throttled`.
If it doesn't output throttled=0x0, then it's either the power supply or your SSD is using too much power (this can only be the case if you're not using the recommended hardware).

#### My Umbrel node doesn't boot. What can I do?
Do you have connected anything to the GPIO pins?
If yes, try to unplug it and reboot the RPi by unplugging the power supply and then plugging it back in.

#### I can't access the dashboard at umbrel.local. What can I do?
Check if your router detects your node
If it does, try to access it with the IP address directly.
If your router doesn't detect the node, either you ethernet cable isn't plugged in correctly or the node doesn't boot.
If you think the ethernet cable isn't the issue, follow the answer of the previous question.
If you can't access the dashboard via the IP address either, try to disconnect the drive from the Raspberry Pi and plug it into the other USB port.
Then SSH into your node and run: `sudo systemctl start umbrel-external-storage`.
After you've run the command, wait for two minutes, then run `sudo systemctl status umbrel-external-storage`.
If the output of that command contains "Exiting the mount script without anything", the drive is connected wrongly.
If the output doesn't contain this text, run `sudo systemctl start umbrel-startup`.
You should now be able to access the dashboard.

#### What are the SSH username and password?
The username is `umbrel`, the password is `moneyprintergobrrr`.

#### I want to connect to my node using ...... over my local network, but it doesn't work. How can I fix this?
If you want to connect to your Umbrel over the local network just replace your onion domain with umbrel.local for any of the connection strings.

-----

I will extend this troubleshooting guide constantly with findings that have been or will be reported in the issues section.
