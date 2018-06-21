---
author: Kent Hua
comments: true
date: "2016-08-09T17:00:00Z"
modified_time: "2016-08-11T17:00:00.000-07:00"
tags: [open source, oss, smartthings, home assistant, ha, hass, homebridge, mqtt, bridge, lirc]
title: Home automation with a series of open source tools and a smart home hub
---

So for the past few years I've managed with just a smartthings hub gen 1, prior to the acquisition by Samsung.  In an effort to try and present as much information of what's happening in the household, I've mounted old smartphones and tablets on the wall.  Initially I ventured with [SmartTiles](http://www.smarttiles.click/), pretty cool way to show and act upon information, but I was limited to just smartthings for the most part.  Everything else would require writing or integrating additional device types.  Not to mention, no one ended up looking at the screen for any updates.  

Many months passed by and the next idea was to integrate Siri into the mix.  Well HomeKit is still not something that Apple has pushed very aggressively.  Maybe with iOS 10 and the dedicated HomeKit app before things take off.  I have a raspberrypi 1 B and a raspberrypi 2 lying around not getting much use.  So on the pi2 I installed [homebridge](https://github.com/nfarina/homebridge) which is a cool app that will integrate my smartthings hub with HomeKit.  Homebridge provides numerous amount of plugins to integrate other components.  I've chosen to use the following plugins [homebridge-smartthings](https://www.npmjs.com/package/homebridge-smartthings), [homebridge-wol](https://www.npmjs.com/package/homebridge-wol) and [homebridge-homeassistant](https://www.npmjs.com/package/homebridge-homeassistant).  So now that I have homebridge up and running, I tried a couple of apps that are HomeKit enabled, [Eve](https://itunes.apple.com/us/app/elgato-eve/id917695792?mt=8) seems to be the one that is most usable from my perspective.  I used the wol function to startup and shutdown my home NAS server.

Next comes my discovery of [home-assistant](https://home-assistant.io/).  This is an open source solution which can essentially tie everything I have together.  Lots of components to talk about, but at it's core I have the [smartthings-mqtt-bridge](https://github.com/stjohnjohnson/smartthings-mqtt-bridge) which ties where most of my zigbee/z-wave devices.  Leveraging mosquitto, the mqtt broker, I am able to have a smartthings device type publish messages to the bridge and then to mqtt.  Home-assistant will then pick it up from the mqtt broker.  Actions performed on home-assistant will be pushed back the other way.  So now I have quite a bit of information flowing into my home-assistant app.  

After all this, the next project was to do what my legacy Logitech Harmony remote can't, integrate with things that aren't infrared.  Sure there was an option to go for the Logitech harmony hub, but what fun would that be.  I still had an extra rpi lying around so I bought an [IR remote shield](http://store.openhapp.com/infrared-shield-for-raspberry-pi/) which connects to my pi's GPIO pins.  I now have an IR receiver and transmitter.  I used the receiver to record my remote commands via lirc, because the lirc database is a bit dated and old.  So now I have my TV, receiver, apple tv and fan IR remotes recorded.  The transmitter will then be used to send commands as part of my home automation routines.  So how does home-assistant talk to my lirc, well I have [lirc_node](https://github.com/alexbain/lirc_node) which will expose lirc in node form and then [lirc_web](https://github.com/alexbain/lirc_web) to provide a REST and Web interface, leveraging lirc_node.  I used home-assistant's REST component to communicate with the lirc_web app.

With all this in place, I can turn on a switch to give power to my receiver and then have the IR transmit turning on the TV, receiver and maybe even light depending on the time of day.

So what do I have hooked up:

* RPi 
  - lirc
  - lirc_node
  - lirc_web

* RPi2
  - homebridge
  - home-assistant
  - smartthings-mqtt-bridge
  - mosquitto
  
* lirc
  - Sony TV Remote - RM-YD012 
  - Onyko Receiver Remote - RC-764M
  - Apple TV - A1294
  - Bionaire Fan Remote
   
Here is a link to my [github](https://github.com/kenthua/homeautomation) repository which contains untested ansible playbooks to install and configure the components.  It also contains the configurations for my devices.  Even for just the lirc remote configurations it's a bonus.


