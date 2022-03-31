---
layout:     post
title:      Raspberry Pi Zero Ethernet Gadget on Linux
date:       2022-03-30 21:21:21
summary:    Get SSH access to your Raspberry Pi Zero Ethernet Gadget on Linux 
categories: raspberry pi zero ethernet gadget linux
---

_![Raspberry pi Ethernet](/images/pi-ethernet.jpg)_

This guide is meant as an extension to the ["Relaying internet access to the Pi Zero 1.3 over USB"](https://github.com/SeedSigner/seedsigner/blob/main/docs/usb_relay.md) document written by the [SeedSigner](https://github.com/SeedSigner) team.

---

The following steps guide you through the necessary commands and configuration changes needed for the creation of a Rpi Zero v1.3 Ethernet Gadget, specifically on Linux.

## Grab a cup of :coffee: and let's get started! 

__1.__ Make sure you have your MicroSD card flashed with the image given at the start of the [Manual Installation Instructions](https://github.com/SeedSigner/seedsigner/blob/main/docs/manual_installation.md). This can be done using the __disks__ tool. Select the drive that corresponds to your memory card, click on the three vertical dots, then _Restore Disk Image_. __Don't eject the SD card after completion!__

__2.__ Follow the steps detailed [here](https://github.com/SeedSigner/seedsigner/blob/main/docs/usb_relay.md#relaying-internet-access-to-the-pi-zero-13-over-usb). Stop after editing the _cmdline.txt_ file. 

__3.__ Set a static IP address for your USB Network device by configuring your address in Raspbian via dhcpcd. To do this, you need to edit the _dhcpcd.conf_ file:

{%highlight bash %}

cd rootfs/etc
sudo nano dhcpcd.conf

# Add the following lines at the end of the file.
# For the "routers" and "domain_name_servers" addresses use the internal IP address of your HOST machine, meaning the device you will connect your rpi0 to.
# For the "ip_address" use the next available address on your network.

interface usb0
static ip_address=192.168.7.2
static routers=192.168.7.1
static domain_name_servers=8.8.8.8 8.8.4.4

{% endhighlight %}

__4.__ Eject the MicroSD card and insert it into you rpi0. The rest of the commands will be executed on our <span class="red">HOST</span> machine.

__5.__ Run `ifconfig` and take note of the output. We will use this to find our rpi0 Interface name later on.

__6.__ Connect your rpi0 to your machine and wait for ~1 minute.

__7.__ Run `lsusb` to see if your rpi0 is properly connected. Look for `Ethernet/RNDIS Gadget`.

__8.__ Run `ifconfig` again and notice any changes compared to the last time you ran the command. The USB interface should begin with _en_, looking something like `enxaef9489ed5c4`. Otherwise, look for `usb0` or `usb1` and so on.

__9.__ Next we need to configure our local computer's USB ethernet to use the IP address of our HOST. 

__Important:__ Make sure to substitute the IP address and the Interface name with the correct ones.

{%highlight bash %}

sudo ip a add 192.168.7.1/24 dev enxaef9489ed5c4
sudo ip link set dev enxaef9489ed5c4 up

{% endhighlight %}

__10.__ You should now be able to ssh into your Raspberry Pi using the address 192.168.7.2 like this:

{%highlight bash %}

ssh pi@raspberrypi.local

# password = raspberry

{% endhighlight %}

### Hurray! 

We can now talk to our rpi0!
The last thing we have to do is give it access to the outside world, not just us.

__11.__ Enable IP forwarding and masquerading on your <span class="red">HOST</span>:

{%highlight bash %}

# For this you will need the interface name of your internet connected interface.
# ifconfig will once again show you your interfaces.
# Copy the name of the one through which you are connected to the internet.
# Mine shows up as wlp4s0, which corresponds to my wifi card.

sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
sudo iptables -t nat -A POSTROUTING -o wlp4s0 -j MASQUERADE

{% endhighlight %}

__12.__  To check if you were successful in your last step, you need to `ping` a website. SSH into your rpi0, like you did in __step 10__, and:

{%highlight bash %}

ping -c 5 google.com

{% endhighlight %}

If the response you get looks something like this: 

{%highlight bash %}

PING google.com (142.251.36.174) 56(84) bytes of data.
64 bytes from muc12s11-in-f14.1e100.net (142.251.36.174): icmp_seq=1 ttl=117 time=29.2 ms
64 bytes from muc12s11-in-f14.1e100.net (142.251.36.174): icmp_seq=2 ttl=117 time=29.3 ms
64 bytes from muc12s11-in-f14.1e100.net (142.251.36.174): icmp_seq=3 ttl=117 time=27.5 ms
64 bytes from muc12s11-in-f14.1e100.net (142.251.36.174): icmp_seq=4 ttl=117 time=26.0 ms
64 bytes from muc12s11-in-f14.1e100.net (142.251.36.174): icmp_seq=5 ttl=117 time=33.9 ms

--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 10ms
rtt min/avg/max/mdev = 26.031/29.178/33.935/2.662 ms


{% endhighlight %}

__Congrats!__ Your Raspberry Pi Zero has now access to the internet. :tada:

Note that these changes are not permanent. You will have to start again from __Step 5__, every time you reconnect your rpi0.




