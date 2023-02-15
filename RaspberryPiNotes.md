# Raspberry Pi Notes

Author: Dakota Turk

Resources such as a list of all websites will be listed throughout but also at the very bottom of this document.

## Overview

Raspberry Pi's are incredible machine. They what are called a micro proccessor and can be used for applications such as embedded systems.

![Picture of Raspberry pis](/images/pi.webp)

They are also heavily used for minor projects or for educational purposes for introducing people to the world of micro computing and hardware/software interaction. As an example, during my time in my undergrad I helped designed a two-way communications device using two raspberry pis, LEDs and a basic frame holding the entire system together made from cardboard. Another project that I have worked on is an individually addressable LED light array.

These little computers can seriously be super powerful as well. They are frequently used as in-home servers for file management, household level VPN protection and even for smart-home furnitures like smart mirrors or smart TVs.

While what we are going to be doing with these pis will be very low-level, the actual applications of these tiny machines are quite limitless.

## Getting started

### Picking the right Pi

If you were to start your very own project with pi and wanted to get your own, the first place to always start is Raspberry Pi's [website (click me)](https://www.raspberrypi.com/). To immediately start looking at what models they currently have and which might be the right one for you go to their [products page](https://www.raspberrypi.com/products/). Clicking on "Learn More" for the product you are interested in will redirect you to a page that details more about the hardware and specifications of the specific board that you chose. It also has a section to actually purchse the board at the bottom.

For reference, the pis that we will be working with are as follows:

- Raspberry Pi 3 (Doug's)
- 3 x Raspberry Pi 4 (Intern's)

There aren't significant differences between these two models, so they work perfect for us. After picking your model, do some searching around on the selected sellers on their website, or find a good kit from onther online seller like Amazon.

### Downloading and onstalling Raspberry Pi OS

Regardless, once you have your Rasberry pi, you're going to need to flash an operating system onto it. In our case, we will be using RasberryPi OS, but in many cases people will use other OS. Personally I prefer ubuntu os on my Pi.

To do this, you're going to need to use what ever SD card your Pi kit included, or buy a seperate 32 GB SD (or lower, not higher as only 32 gigs are supported for the SD card slot on the board) and find the Raspberry Pi Imager software on Raspberry Pi's [website](https://www.raspberrypi.com/software/).

![RaspberryPi Imager picture](/images/rpi-imager.png)

Once running it you will be met with the screen above. Decide which OS you want to use on the left and select it. Plug your SD card into your computer (usually your Pi kit will come with a reader if your machine doesn't have one built in) and then select that storage device using the middle selector. Finally just press the "Write" button and let the software do its thing. Eventually it will finish and you will be able to plug the SD into your Rasberry Pi.

### Boot up

The last and easiest step before diving into the world of Raspberry Pi is to turn your Pi on for the first time! Your Pi should have came with an included power cable (again, usually comes in the kit). If it didn't just find the right one online to use get that. With the newer Raspberry Pis this power cable is usually a type-c cable. For the older generations it is a micro-usb. Also for more advanced set ups and with specific models of the Raspberry Pis, you can even get a power over ethernet adapter to just need the ethernet cable of your home network.

After turning your Pi on it will load into its boot loader and activate your Pi. You'll need to follow any on screen instructions in order to setup your account on your Pi and to begin playing around with it. Thankfully much of these steps have already been done for you. Each of your Pis will be setup with the account details as follows:

- User name: rpifun
- Password: pitime

## Setting up the Pi

The first thing that has been done to the pis, is their initial start up and update process. They have all been logged into with the same exact username and password so that you can each easily get into them. The next setup that was done on them is assigning a static IP. Unfortunately, since out team doesn't have enough mini HDMI to HDMI cords, there is no way that we can get you guys to interact with the pi with a mouse and keyboard directly. Instead, however, we can allow to to connect to them using a static IP address. Thankfully this is really easy to set up. In case the steps aren't clear here and you would like to pull this off yourself, follow this [article](https://magpi.raspberrypi.com/articles/connect-raspberry-pi-4-to-ipad-pro-with-a-usb-c-cable).

### Setting up the Raspberry Pi to use ethernet through USB C

This setup will be preparing ssh and vnc to be usable through the USB C. To learn more about what VNC is a few simple google searches would work. It essentially allows us to render the screen through an ssh terminal. Though we won't be using it here.

- The first thing to do is update/upgrade your pi and make sure its up to date:

``` CMD
sudo apt update
sudo apt full-upgrade
sudo reboot
```

- Next step is to adjust the boot config file:

``` CMD
sudo nano /boot/config.txt
```

- Then you'll want to make sure the following lines are uncommented (delete the #'s):

``` CMD
framebuffer_width=1024
framebuffer_height=768
```

- Then add the following underneath the ```[all]``` tag:

``` CMD
dtoverlay=dwc2
```

- After this, press CTRL + O, then enter and then CTRL + X, to save and close the file.

- Then open the ```/boot/cmdline.txt``` file:

``` CMD
sudo nano /boot/cmdline.txt
```

- Add this to it and then save and quit:

``` CMD
modules-load=dwc2
```

- Create a blank file for ssh in the ```/boot``` directory

``` CMD
sudo touch /boot/ssh
```

- Open dhcp config file:

``` CMD
suco nano /etc/dhcpcd.conf
```

- Add this line to it, then save and quit with CTRL + O, enter then CTRL + X:

``` CMD
denyinterfaces usb0
```

- Next is to adjust the modules files:

``` CMD
sudo nano /etc/modules
```

- Add this line to the end:

``` CMD
libcomposite
```

- Then CTRL + O, then Enter followed by the CTRL + X again to save and exit.

- Next we will install some net tools to allow us to set up the static IP address stuff:

``` CMD
sudo apt install dnsmasq -y
```

- Once that finishes, create a "usb" file:

``` CMD
sudo nano /etc/dnsmasq.d/usb
```

- Then add the following code into it:

``` CMD
interface=usb0
dhcp-range=10.55.0.2,10.55.0.6,255.255.255.248,1h
dhcp-option=3
leasefile-ro
```

- Now we setup our static IP adress. Edit the interface for usb0:

``` CMD
sudo nano /etc/network/interfaces.d/usb0
```

- Add the following into the file and then save and quit again:

``` CMD
auto usb0
allow-hotplug usb0
iface usb0 inet static
    address 10.55.0.1
    netmask 255.255.255.248
```

- Now we will enable some settings for VNC. A more comprehensive look at this can be found [here](https://www.hardill.me.uk/wordpress/2019/11/02/pi4-usb-c-gadget/):

``` CMD
sudo mousepad /root/usb.sh
```

- Then copy and paste the code found at the url listen above the last command into the file and save and exit:

``` CMD
#!/bin/bash
cd /sys/kernel/config/usb_gadget/
mkdir -p pi4
cd pi4
echo 0x1d6b > idVendor # Linux Foundation
echo 0x0104 > idProduct # Multifunction Composite Gadget
echo 0x0100 > bcdDevice # v1.0.0
echo 0x0200 > bcdUSB # USB2
echo 0xEF > bDeviceClass
echo 0x02 > bDeviceSubClass
echo 0x01 > bDeviceProtocol
mkdir -p strings/0x409
echo "fedcba9876543211" > strings/0x409/serialnumber
echo "Ben Hardill" > strings/0x409/manufacturer
echo "PI4 USB Device" > strings/0x409/product
mkdir -p configs/c.1/strings/0x409
echo "Config 1: ECM network" > configs/c.1/strings/0x409/configuration
echo 250 > configs/c.1/MaxPower
# Add functions here
# see gadget configurations below
# End functions
mkdir -p functions/ecm.usb0
HOST="00:dc:c8:f7:75:14" # "HostPC"
SELF="00:dd:dc:eb:6d:a1" # "BadUSB"
echo $HOST > functions/ecm.usb0/host_addr
echo $SELF > functions/ecm.usb0/dev_addr
ln -s functions/ecm.usb0 configs/c.1/
udevadm settle -t 5 || :
ls /sys/class/udc > UDC
ifup usb0
service dnsmasq restart
```

- After this then make the file executable using this command:

``` CMD
sudo chmod -x /root/usb.sh
```

- Then we will use crontab to edit the startup options inorder to get the static IP up when the pi starts up. When crontab opens, you will need to select ```1``` and then hit enter to use nano to edit the startup options:

``` CMD
sudo crontab -e
```

- Add the following line at the bottom of this page:

``` CMD
@reboot bash /root/usb.sh
```

- Then finally, shutdown the pi:

``` CMD
sudo shutdown -h now
```

- You can now connect the pi to your computer, iPad or other smart device that has VNC or SSH capabilities and ssh into your pi using this command:

``` CMD
ssh pifun@10.55.0.1
```

- Ensure that you are using a capable USB C cord as well since some cords are incapable.

What assigning a static IP address does, is we can tell our pi to tell any computer that it interfaces with, that its "name" or identification is one singular value. Normally, our routers and computers will assign IP addresses to new machines as it sees fit, but in this case it will get the value directly from the pi instead of making a new one. This allows us to use commads like sftp (secure file-transfer protocol) and ssh (secure shell) in order to interact with our pi. While I will show you what it is like to see the pi and interact with it using a mouse, keyboard and monitor, our main interaction will be happening over an ssh shell. I will show you how to set this up in vscode later as well so please ensure that you have downloaded vscode.

## Resources

- [Set up](https://www.raspberrypi.com/tutorials/how-to-set-up-raspberry-pi/)
- [PoE Hat for specific Pi models](https://www.raspberrypi.com/products/poe-hat/)
