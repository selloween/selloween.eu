+++
title = "How to install i3 on Debian 9"
date = "2018-12-26"
author = "Selwyn"
cover = "hello.png"
description = "A step by step guide on how to install i3 on top of a minimal Debian 9 setup"
+++

## Debian network install

### Requirements
* AMD64(x86-64) based processor
* Ethernet cable plus  a stable internet connection
* USB drive with a minimum of 300MB disk space

### Download the image
Download the network install image from here: https://www.debian.org/devel/debian-installer/
Be sure to download the netinstaller image for AMD64 systems.

### Create a bootable USB drive

#### On Windows
Download Rufus: https://rufus.ie

#### On Linux
Open a terminal and use "fdisk"

```bash
$ sudo fdisk -l
```

Identify the correct disk name according to the naming pattern of
"/dev/sdx". In the example below the disk name is "/dev/sdb".

```
Device     Start       End   Sectors   Size Type
/dev/sdb1   2048 240252927 240250880 114.6G Microsoft basic data
```
##### Flash the drive
Use the "dd" command to flash the usb drive

```bash
$ sudo dd bs=4M if=debian-9.6.0-amd64-netinst.iso of=/dev/sdx status=progress conv=fsync
```

## Base installation
Restart and before the welcome screen appears press "Enter" or "F12" - the key might
vary depending on your hardware manufacturer. In the boot options menu choose boot from USB.

### Graphical Installation
On the main Debian installation screen choose graphical installation.

![Debian Graphical Installation](/img/posts/2018/12/26/01-debian-graphical-install.png)

### Post installation
After you login for the first time switch to root and enter the password when
prompted. (Don't forget appending " -" after "su")

```bash
$ su -
```

#### Update system

```bash
# apt-get update && apt-get upgrade
```

#### Install sudo
```bash
# apt get install sudo
```
#### Add user to sudo group
The user must be added to the sudo group to have sudo privileges.
```bash
# usermod -aG sudo username
```
Switch back to user
```bash
# su username
```
## Install a text editor and git
One of the most important tools is a text editor. You will need one for the next steps.
I suggest installing vim, but feel free to install any other. (e.g. nano)

```bash
$ sudo apt-get install vim
```


## Adding non-free and contribution repositories
If you want to install certain common software such as Chromium, Firefox etc. you need to 
add them to the "sources.list". Depending on your hardware you might need proprietary firmware for
e.g. WiFi to function.

```bash
$ sudo vim /etc/apt/sources.list
```

Adjust the entries according to the example:

```
deb http://site.example.com/debian stretch main contrib non-free
deb-src http://site.example.com/debian stretch main contrib non-free
```
Update packages

```bash
$ sudo apt-get update && sudo apt-get upgrade
```

More info on sources list: https://wiki.debian.org/SourcesList

## Upgrade to Debian Testing or Debian Sid
Here would be a good point to upgrade to "Debian Testing" or "Debian Sid" if you wanted to.
I have written a seperate guide on how to do this here:
This step is optional and can be done later on.

## Network management

### Install Network Manager

```bash
$ sudo apt-get install network-manager
```

### Intel WiFi firmware
To enable support for Intel 802.11n devices on Debian systems you will have to install the intel
wifi firmware package. Make sure that the non-free option is added to "/etc/apt/sources.list" as
shown above.

```bash
$ sudo apt-get update && apt-get install firmware-iwlwifi
```
### Connect to WiFi

Run "nmtui" and activate a connection (Screenshot)

```bash
$ nmtui
```
If you can connect successfully to a WiFi network but can't connect to the internet, you might need
to reboot.

## Desktop installation

### Install X-Server
To get i3 running you need to first install Xserver.

```bash
sudo apt-get install xorg
```

###  Install i3 and dmenu
Now install i3 and dmenu. dmenu is a handy and lightweight  application launcher.

```bash
sudo apt-get install i3
```

Note: i3 on Debian  comes prepackaged with "suckless tools" preinstalled including "dmenu", a
lightweight application launcher, "dunst", a notification daemon, and
"i3lock", a simple screen locker. More info on "suckless tools": https://suckless.org/

### Login Managment

### Bash Login
To keep the system as minimal as possible I suggest using the bash login and configuring it so that
i3 starts after login. If you prefer having a graphical login screen head to the next step.


```bash
$ touch  ~/.bash_profile
```

Then add following line by pasting this command

```bash
$ printf "if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then\n\texec startx\nfi" ~/.bash_profile

```

### GUI Login Manager
Instead of the bash login you can install a graphical Login Manager e.g. LightDM

```bash
$ sudo apt-get install lightdm
```

After reboot you will be welcomed by a graphical login interface.
Login and you should be prompted by the config wizard of i3!

Let i3 generate the default config and on the next prompt choose the mod key you want to use.
(Windows Key or Alt)























## Get WiFi working
Depending on your hardware you might need to install additional firmware.
On my ThinkPad T480s I had to install proprietary firmware to get proper WiFi.









#### 














## Base Installation

## Update

Switch to the root user as sudo is not setup yet.
Type `su -` and enter the root password.

```bash
$ apt-get update && sudo apt-get upgrade
```

## Text-editor

Install a text-editor. I suggest using vim but feel free to install any other editor of choice.

```bash
$ apt-get install vim
```

## Sudo

Install sudo

```bash
$ apt-get install sudo
```

Add the user to the sudo group.

```bash
$ usermod -aG sudo username
```

Leave the root session

```bash
$ exit
```

## Sources List

### Non-free packages

Depending on your software needs and hardware you might need non-free packages.

#### Adjust sources.list

Add non free-packages repositories by editing the sources list located in '/etc/apt/sources.list'
Add the terms 'contrib' and 'non-free' to each source entry according to the example bellow.

```bash
$ sudo vim /etc/apt/sources.list
```

### Debian Testing (optional)

Debian testing is the current development state of the next stable Debian distribution.
More info on Debian Testing: https://wiki.debian.org/DebianTesting

Make sure your current stable version is up to date.

```bash
sudo apt-get update && sudo apt-get upgrade
```

* Edit sources.list as mentioned, except replace the distribution name to 'testing' as in example bellow.
* Edit your /etc/apt/sources.list file, changing 'stable' (or the current codename for stable) to 'testing'
* Remove or comment out your stable security updates line(s) (anything with security.debian.org in it).
* Remove or comment out any other stable-specific lines, like *-backports or *-updates.

```
deb http://site.example.com/debian sid main contrib non-free
deb-src http://site.example.com/debian sid main contrib non-free
```

Now update and upgrade the distribution

```
sudo apt-get update & sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### Debian Sid (optional)

Sid is the unstable branch of Debian and can be compared to a bleeding-edge rolling release distribution.
Note that using Sid comes with risks as things can break. Do not ever use sid on a server!
More info on Debian Sid: https://www.debian.org/releases/sid/

Make sure your current stable version is up to date.

```bash
sudo apt-get update && sudo apt-get upgrade
```

* Edit sources.list as mentioned, except replace  the distribution name to 'sid' as in example bellow.
* Edit your /etc/apt/sources.list file, changing 'stable' (or the current codename for stable) to 'sid'
* Remove or comment out your stable security updates line(s) (anything with security.debian.org in it).
* Remove or comment out any other stable-specific lines, like *-backports or *-updates.

```
deb http://site.example.com/debian sid main contrib non-free
deb-src http://site.example.com/debian sid main contrib non-free
```


Now update and upgrade the distribution

```
sudo apt-get update & sudo apt-get upgrade
sudo apt-get dist-upgrade
```








## Nonfree packages










transmission-cli
udiskie
intel-microcode
pcmanfm
mpv
htop
git
nitrogen
gparted
gpart
arandr
firefox
qutebrowser
ranger
network-manager
network-manager-gnome
firmware-iwlwifi
exec --no-startup-id nm-applet
exec --no-startup-id nitrogen --restore
tlp
service tlp start
docker
docker-compose
xbacklight

# Sreen brightness controls
bindsym XF86MonBrightnessUp exec xbacklight -inc 20 # increase screen brightness
bindsym XF86MonBrightnessDown exec xbacklight -dec 20 # decrease screen brightness


acpi
acpid
pulseaudio
compton

/etc/X11/xorg.conf.d/20-intel.conf

Section "Device"
    Identifier  "Intel Graphics" 
    Driver      "intel"
    Option      "Backlight"  "intel_backlight"
EndSection

https://wiki.archlinux.org/index.php/backlight#xbacklight


reboot
