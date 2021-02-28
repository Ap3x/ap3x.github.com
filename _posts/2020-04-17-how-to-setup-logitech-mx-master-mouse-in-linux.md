---
title: "How to Setup Logitech MX Master Mouse with Linux"
date: 2020-04-17 1:58:00 -0500
categories: [How To, Setup Logitech MX Master Mouse with Linux]
tags: [logitech, how to, shortcut]
---

So I have a Logitech MX Master and I really enjoy the ergonomic feel of the mouse. The mouse is fitted with a few spare buttons on the side of the mouse and it comes in handy a lot, but the issue I had was that Logitech provides only a program for Windows machines to download and no support for Linux. So with a little linux knowledge we can map these keys to useful commands on our linux machine.

The command that I find very useful is:
> gnome-screenshot -a
This command takes a screenshot of an region on your screen.

So while I was studing for the OSCP I wanted a way to screenshot my screen or whatever is that I was doing on the fly without press some sort of keyboard shortcut.

So in order to figure out which keys the MX Master is sending to the system we can use a tool called xbindkey. This binary lets to map buttons to commands and record what keys are pressed. 

So first lets download it:
> sudo apt install xbindkeys

Next we want to see what keys are sent to the system when we press the side button on the MX Master.

So run the following command:
> xbindkeys --multikey

A little window should pop up and now you can press the side button on the MX Master mouse. When you press the button you should get some similiar to what is displayed below.

<img src="/assets/img/post/xbindkey.png" alt="Desktop View" style="width: 70%" >

As you can see that xbindkey recognized that Tab + Left Alt + Left Control were sent.

Now that we know the keys that are sent we can add that to our shortcuts in the settings in Ubuntu.
Go to `Settings -> Devices -> "Keyboard Shortcuts"` 
> NOTE: You may have to disable the shortcut "Switch system control"

Scroll to the bottom and click the "+" and come up with a name for the command. Then in the command field put:

> gnome-screenshot -a 

Next click "Set Shortcut" and click `Left Control + Left Alt + Tab` on the keyboard and set shortcut.

<img src="/assets/img/post/keyshortcut.png" alt="Desktop View" style="width: 70%" >

Now you should be able to screenshot a region with you MX Master mouse using the side button.