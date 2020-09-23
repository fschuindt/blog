---
layout: post
title: "Build showcase: My wireless Raspberry Pi powered \"science station\" for controlling the telescope mount, DSLR camera and all future gear"
categories: [IT, Astronomy]
---

*This post was originally published on my [old blog](https://boredprogrammer.postach.io/post/build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear) dedicated to amateur astronomy.*

I'm really happy to announce that I finished to building/setting up what I'm calling my "science station". It's a plastic box with a single board computer, a wifi router and some other stuff that allows me to control physical devices like equatorial telescope mounts and DSLR cameras remotely from my notebook with no wires in between.

![01]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200730_171105.jpg)

I believe the idea here is really similar to products like [SkyFi](https://skysafariastronomy.com/skyfi-3-professional-astronomy-telescope-control.html) and [StellarMate](https://www.stellarmate.com/) but with a DIY element. Also I'm planning to later hook it up with non-astronomy related things, like [SDR](https://en.wikipedia.org/wiki/Software-defined_radio) devices.

The main element of my build is a Raspberry Pi 4 with 4GB RAM. It's running Manjaro ARM Linux on a 64GB SD card. I've also compiled and installed myself all the [INDI Core libary](https://github.com/indilib/indi) as well as the [INDI 3rd Party libraries and drivers](https://github.com/indilib/indi-3rdparty), and installed the [INDI Web Manager](https://github.com/knro/indiwebmanager) using Python's pip. With that the Pi is already able to talk to dozens of astronomy devices like cameras, focusers, filter wheels, etc.

Now with the INDI Web Manager configured as a systemd service (as well with OpenSSH), I can say that the device is ready to roll. And I'll be using it to control my Meade LX85 EQ mount and my wife's Canon EOS 600d DSLR camera. Focus is still manual, but I'm OK with that. And either way it's just a matter of getting a focuser later on the road.

Inside the plastic box you can also see a wifi router, and that's only for when I'm outside home. When I'm home, the Pi is programmed to connect to my home's wifi connection, and my notebook will do the same, so that wifi router inside the station is just ignored. With that I can connect the notebook and station together and also enjoy an internet connection on the notebook. When I'm outside however, I can just connect to the 2.4GHz session that this router is broadcasting, which will allow me to connect the Pi <--> Notebook, but no internet, since I'm outside anyway.

All other stuff is basically power adapters on a single extension (to power everything using a single outlet terminal) and a handmade cable management board. So that the cables won't be entangling inside.

In total it has 4 cables running from/to the station.

Running in:
- Main power cable (from the wall outlet)

Running out:
- EQ mount power cable
- EQ mount serial data cable
- DSLR USB cable

All these cables can be connected close to the telescope/mount, so the station lives close to the telescope itself. But no cables on the notebook, all is connected using wifi.

On the notebook, the software I'm running to control everything is [KStars](https://edu.kde.org/kstars/) with [Ekos](https://www.indilib.org/about/ekos.html) on a Manjaro Linux workstation.

The next step is to configure a wired XBox-like joystick on the notebook to control the mount and later to set up my [RTL-SDR](https://www.rtl-sdr.com/) with a VHF antenna on the station itself.

![02]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200730_171508.jpg)

![03]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200730_170935.jpg)

![04]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200730_171003.jpg)

**Update:** It's now working controlled by a USB joystick. The joystick is connected directly on the notebook.

![05]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200801_195434.jpg)

![06]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200801_192154.jpg)

![07]({{ site.url }}/images/2020-07-31-build-showcase-my-wireless-raspberry-pi-powered-science-station-for-controlling-the-telescope-mount-dslr-camera-and-all-future-gear/IMG_20200801_192751.jpg)
