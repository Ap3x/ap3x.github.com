---
title:  "Infection Monkey - Securing My Network"
date: 2020-08-10 11:48:01 -0400
categories: [Homelab , Security]
tags: [infection, monkey, blue, team, linux, hardening]
---

# DEFCON Safe Mode 2020 - Infection Monkey
During this week of DEFCON Safe Mode I listened to a lot of fantastic talks from new techniques about Domain Hiding (Domain Fronting), Red Team CCDC techniques, and this neat tool called Infection Monkey. You can find the talk at this link https://youtu.be/gOS1c375Hbg and more about the tool at their website here: https://www.guardicore.com/infectionmonkey/. Essentially Infection Monkey is an open source tool that simulates a breach and attack simulation assessing the resiliency of environments. The tool offers a lot of cool features such as testing for network segmentation, lateral movement, common exploits, and even allowing you to do more advance customization. The tool also give you a security report in three different formats the Infection Monkey format, Zero Trust Report format, and the ATT&CK Report.

## Lets Give It A Try
After watching the video I decided to try this out on my own homelab environment and see what it could find. If you've read my previous post about my homelab you would know that it is mainly comprised of Linux containers/VMs so I didn't manage to test the Windows feature of this tool. I'm not gonna walk through how to setup Infection Monkey just because how trivial it is to setup. When I setup the tool I used a Ubuntu 18.04 container and downloaded the DEB package straight from the site and installed it.

## Configuration
After you login and setup an account you will be prompted to configure the "Monkey" this is supposed to represent the attacker on the network. I provided some of the users and passwords that I use on machines in my network and I didn't provide any SSH keys just to see which machines I may have misconfigured setting up SSH. I also provided some of the subnets of my different VLANs to test for proper network segmentation and scanning of devices on each subnet. Under the Internal tab within the configuration I listed some of the obscure port numbers I use for some services to use for scanning. Then I left everything else default. The command and control server for the attacker is called the Monkey Island.

<img src="/assets/img/post/2020-08-10-infection-monkey---securing-my-network/MonkeyIsland2.png" style="width: 75%" >

## Let the Monkey Run Wild
I began the attack by running the Monkey on the Monkey Island Server which is the server hosting the web GUI or the command and control server. The Monkey Island was placed in the master VLAN which has access to all the VLANs. The reason I started the attack in the master VLAN was because I knew that the monkey would propagate to all the subnets and test all the servers. This will also help because if the monkey is able to laterally move to one of the subnets then it will test and see if it can hop to another VLAN or subnet.

<img src="/assets/img/post/2020-08-10-infection-monkey---securing-my-network/MonkeyIsland1.png" style="width: 75%" >

As you can see the monkey managed to scan a lot of devices on the different subnets and if you notice the little virus icon on the bottom right of each of the computers shows that it was successfully exploited and managed to gain a shell. Many of the machines that were infected due to giving a set of passwords for the monkey to test SSH login. In the report it also details info on how it broke in what it managed to find on that machine.

Below are some pieces of the zero trust report feedback for my network. 

<img src="/assets/img/post/2020-08-10-infection-monkey---securing-my-network/MonkeyIsland4.png" style="width: 75%" >
<img src="/assets/img/post/2020-08-10-infection-monkey---securing-my-network/MonkeyIsland3.png" style="width: 75%" >

At the top of the ATT&CK Report is shows the sections of the framework that we successfully executed. You can go through each of the sections and they report will show you the techniques used to attack and breach the system.
<img src="/assets/img/post/2020-08-10-infection-monkey---securing-my-network/MonkeyIsland5.png" style="width: 75%" >

Near the bottom of the report Infection Monkey offers mitigation help for the sections that were successful as seen in the photo below.

<img src="/assets/img/post/2020-08-10-infection-monkey---securing-my-network/MonkeyIsland6.png" style="width: 75%" >

## Take Away
In all this is a great tool from the setup being so simple to the end printing out the report. Although there are a few tweaks I wish were fixed or changed on Infection Money and that is printing the report. There is an option to print the report, but it seems like it just prints the webpage in the current state and doesn't show the expanded view every subheading. I think in the future I will do a more customized and advance configuration to truly test my network. At the time of this posting I will have fixed all the issues that are listed in the report for my own network safety. I also wanna spin up my AD practice lab with a couple Windows computers to test Infection Monkey and see what it can find.