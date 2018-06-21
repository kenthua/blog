---
author: Kent Hua
comments: true
date: "2017-04-10T11:00:00Z"
modified_time: "2017-04-10T17:00:00.000-07:00"
tags: [smartthings, homeassistant, ha, automation, home, relay, sensor]
title: Home Automation - Garage Door
---

It's been a long time coming, but I've finally managed to automate my garage.  Thus far I've had my home automation setup give me information about my home, turning lights on and off, and adjusting the temperature of my nest.  I never got around to automating the garage because it involved some wiring.  So what did I acquire and setup?

* [Extension cord power block](https://www.amazon.com/gp/product/B01FX6JSGC/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
* [GoControl Z-Wave Isolated Contact Fixture Module - FS20Z-1](https://www.amazon.com/gp/product/B00ER6MH22/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
* [Gardner Bender 10-004 WireGard Screw-On Wire Connector, 18-10 AWG, Yellow](https://www.amazon.com/gp/product/B002YE7W42/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
* Computer universal power cable with ground 
* SmartThings relay configuration
* home-assistant integration

The extension cord was because my garage door opener only had 1 power outlet exposed in the area, so I had to get a power block to expand it to 2+ outlets.  One for the garage door opener and the other for the relay module.  

The wire nut was for the wiring because this relay module came with a bunch of wires sticking out.  The power cable was connected the white, green and black wires coming out of the relay, as noted in this [blog entry](http://blog.smartthings.com/how-to/how-to-openclose-your-garage-door/).  The blue wires were to be connected to my garage door opener.  The blue wires are 12AWG so I had to get some wire nuts capable of twisting at least 12AWG.  The power cable was only around 16-18 AWG, so I could have gotten smaller.  Now the blue wires being 12AWG are a bit large to fit into my garage door opener push slot, so I had to get some smaller wires to essentially reduce the wire size.  You are to fit the blue wires into where the garage door button is wired.  There is no +/- since it's just a open/close circuit that the garage door opener is looking for.  The blue wires are 12AWG because it's meant to handle 20AMPs.  Alternatively one could also just shave down the wires to a smaller size.

The relay is sitting on top of my opener, though I've seen the relay can also fit into a typical electrical outlet box.

So once it was all wired up, now onto the software side.  I had to configure the proper device type in smartthings, I tried different ones as recommended by others for other applications, but this one worked, "Z-Wave Virtual Momentary Contact Switch."  So within smartthings it's treated as just a single button, so there is no "on/off" essentially.  

I also have a smartthings multi sensor to detect whether the state of the door is open or closed.

Once I got it all up and running in smartthings, time to get it properly working in Home Assistant.  I've only ventured into the high level portions of automation/scripting within HA.  I'm sure I can do much more, but just to get it work, I had to define an mqtt switch and a script in order to reset the switch each time it's pushed.  Since it's a switch it requires on/off.  The [script, garage_door_button, ](https://github.com/kenthua/homeautomation/blob/master/homeassistant/includes/script.yaml) I created has a delay and then sets the state back to off so it doesn't get stuck in the on state.

So what did I get after all this?  I can now open/close my garage with a push of a button remotely.
