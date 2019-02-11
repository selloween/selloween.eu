+++
title = "How to install Magic the Gathering Arena on Debian Stretch"
date = "2019-01-16"
author = "Selwyn"
cover = "mtg-arena-on-linux.png"
draft = "false"
description = "A tutorial on how to install MTG Arena on Debian using PlayOnLinux. "
+++


## Play MTG Arena on Linux

In this tutorial I will show you how to install `Magic the Gathering Arena` on  Debian Stretch using PlayOnLinux.

### My Setup
I tested the game on following system and am amazed how well it works.

* Thinkpad T480s
  * Intel i5 8250U
  * Intel UHD 620 onboard GPU
  * 12 GB DDR4 Ram
  * Game Resolution of 1920*1080
  * Low graphic settings

Depending on your hardware specs, the graphics settings will vary. Having a dedicated GPU will
benefit gameplay. I am content playing on low settings without the need of dedicated graphics.

## PlayOnLinux

PlayOnLinux is a graphical frontend for Wine with which you can install Windows applications on Linux. Wine emulates the Windows runtime environment by translating Windows system calls into POSIX-compliant system calls.

## Download MTG Arena

First download MTG Arena Windows executable from the official website: [https://magic.wizards.com/en/mtgarena](https://magic.wizards.com/en/mtgarena) 


## Install PlayOnLinux on Debian Stretch

	
On Debian Stretch you have to add the contribution repository by adding it to `/etc/apt/sources.list`.

* Edit `/etc/apt/sources.list` with the text editor of choice. Make sure to add `contrib`to each source entry as demonstrated below.

```bash
deb http://deb.debian.org/debian/ stretch main non-free contrib
deb-src http://deb.debian.org/debian/ stretch main non-free contrib
```

* Update system packages:

	`sudo apt-get update && sudo apt-get upgrade`

* Install PlayOnLinux:

	`sudo apt-get install playonlinux`
	
### Other Linux distributions

PlayOnLinux packages are available for several Linux distributions including Ubuntu, Arch Linux and Fedora.
You can download the appropiate package here:   [https://www.playonlinux.com/en/download.html](https://www.playonlinux.com/en/download.html) or clone the git repository for the latest development version and build from source.

`git clone https://github.com/PlayOnLinux/POL-POM-4`

## Installation

* Start PlayOnLinux

### Manage Wine Versions

* First nagigate to  `Tools`tab  in the top menu bar and select `Manage Wine Versions`

![](/img/playonlinux_00.png) 

* Click on `Tools` in the upper navigation bar. From the dropdown select `Manage Wine Versions` and add `3.20`. Do this for `x86` and `amd64`.

### Install the program

Now we will create the Windows virtual drive and install Magic the Gathering Arena!

![](/img/playonlinux_01.png) 

* Click the `Install` Button
<br/>
<br/>

![](/img/playonlinux_02.png) 

* In the install menu choose `Install a non-listed program`
<br/>
<br/>

![](/img/playonlinux_03.png) 

* Click on `Next`
<br/>
<br/>

![](/img/playonlinux_04.png) 

* Select `Install a program in a new virtual drive`
<br/>
<br/>

![](/img/playonlinux_05.png)

* Name the virtual drive e.g. MTGA
<br/>
<br/>

![](/img/playonlinux_06.png)

* Select `Use another version of Wine`
* Select `Configure Wine`
<br/>
<br/>

![](/img/playonlinux_07.png)

* Select `3.2`
<br/>
<br/>

![](/img/playonlinux_08.png) 

* Select `64 bits windows installation`
<br/>

   
![](/img/playonlinux_09.png)    
![](/img/playonlinux_10.png)    

* Install `Mono` and `Gecko` when prompted.
<br/>
<br/>

![](/img/playonlinux_11.png)    

* When the Wine configuration window appears, make sure that `Windows 10`is selected under the
* `Applications` tab. This is crucial as the  installer will crash if any other version is selected!
  
 * Then navigate to the  `Libraries` tab and select `d3dx11_43` to install DirectX 11. Click on `Add` and then on `Apply`.

* Press the `OK` button to close the configuration window.
<br/>
<br/>
<br/>
![](/img/playonlinux_12.png)    

* Point to the `MTGAInstaller.exe` Windows executable installation file and press `Next`
<br/>
<br/>
![](/img/playonlinux_13.png)   
* Install MTG Arena! Yaaaaay!
<br/>
<br/>

### Final steps

After the installation of Magic the Gathering Arena has finished the Installation Wizard will guide you to some further few steps.

![](/img/playonlinux_14.png)  

* Select `MTGA.exe`to add a shortcut.
* Then select `I don't want to make another shortcut`.
 <br/>
 <br/>

![](/img/playonlinux_15.png)   

* Name the shortcut e.g. MTGA.
 <br/>
 <br/>

![](/img/playonlinux_16.png)   

### Set the Video Memory size

  * Before you start the game, head to the `Configure` button (the one with the cog-wheel icon).
  * In the popup window select the newly created `MTGA` Windows virtual drive and select the `Display`tab. 
  * Leave all values at `default` except for `Video memory size`.
  * Set it to a minimum of `4096`. (This is the maximum on my system - it might be higher depending on your system ressources).
  <br/>
  <br/>
### Play the Game
* That's it! 
* Play the game!
<br/>
<br/>
### Notes
* The installation process might crash several times while downloading filess.  If so, don't give up and restart the process. The installer should continue downloading files.
* I highly advice stopping any background applications or activities during installation. My
* installation crashed several times  while surfing the web.


