---
title: "My HomeLab"
date: 2020-04-16 18:55:00 -0400
categories: [HomeLab, Proxmox]
tags: [homelab, networking, proxmox, raspberry pi, octopi]
---
In my homelab I have a couple servers and networking gear that I use for building out different network environments to learn about pentesting, and a little about blue team tactics. I also use my server, Nebula to host services that I use frequently use and that I would consider my "production environment". As for my other server, Pulsar I use it as a "dev environment" to test different tools, pentest, and work on projects.

Below is a more in depth view of my equipment and some of the services I host in my lab.

## Server Photos

<img src="/assets/img/post/rotated-ServerRack.jpg" alt="Desktop View" style="width: 30%" >

## Lab Gear

|Server Name|Equipment|Description|Server Hardware
|:---|:--|:--|:--|
|Pulsar |Dell PowerEdge R520 | Proxmox Virtual Environment     | 24 x Intel(R) Xeon(R) CPU E5-2420; 96Gb DDR3 RAM 
|Nebula |Dell PowerEdge R710 | Proxmox Virtual Environment     | 16 x Intel(R) Xeon(R) CPU E5520; 48Gb DDR3 RAM  
| N/A   |Linksys SRW2024     | Managed Switch                  | 24 port 10/100/1000 Switch 
| N/A   |                    | UnManaged Switch                | 24 port 10/100/1000 Switch 
| N/A   |Raspberry Pi(s)     | OctoPi and MotionEyeOS          | Raspberry Pi A,B+

## Services 

|Server|Service|Description| 
|:---|:--|:---|
|Nebula       | Pi-Hole             | DNS Server for blocking Ads
|Nebula       | Grafana             | View different network and server demographics
|Nebula       | Ubooquity           | A library that holds all virtual books
|Nebula       | TurnKey NFS         | Small storage for quick transfers and temp storage
|Nebula       | ELK Stack           | Monitoring authentications and logs
|Nebula       | OpenVPN             | Used for connecting to my network when I'm away
|Nebula       | FireFly III         | Manage my budget and spending
|Nebula       | MediaWiki           | Wiki store links to all my services interfaces and notes for fixes or info of services
|Pulsar       | Kali                | Pentesting when I dont have my PC available
|Pulsar       | Wekan               | Manage coding projects
|Pulsar       | Pfsense             | Used for the AD network for pentesting
|Pulsar       | Ubuntu WebApp       | Colliagte Cyber Defense Comp. (CCDC) Scoring Engine Server
|Pulsar       | Ubuntu WebApp       | CTFd for building out my own CTF
|Pulsar       | GitLab              | Maintain Coding Projects
|Pulsar       | GrayLogs            | Monitor logs in the AD enviroment
|Pulsar       | Ubuntu Desktop      | Another Coding Enviroment
|Pulsar       | Splunk              | Monitor logs in the AD enviroment
|Pulsar       | Plex                | Currently Not in use
|Pulsar       | Windows10           | Dev Enviroment for coding projects
|Pulsar       | Windows Server 2016 | Active Directory Domain Controller for pentesting
|Pulsar       | Windows10 Client1   | Computer used in the AD forest for pentesting
|Pulsar       | Windows10 Client2   | Computer used in the AD forest for pentesting
|Pulsar       | VARIOUS             | Entire copy of the Colliagte Cyber Defense Comp. (CCDC) network
|Raspberry Pi | OctoPi              | Manage my 3D printer
|Raspberry Pi | MotionEyeOS         | Network camera to monitor my servers


## 3D Printer

I purchased this cool little printer not to long ago and have had nothing but a positive experience from working with it. I usually just print toys, figurines, tools, or print broken parts. The 3D Printer in the photo below is the Creality Ender 3 Pro

<img src="/assets/img/post/3D-Printer.jpg" alt="Desktop View" style="width: 30%" >

There are a few modifications that I added to my printer such as the cable chain on the back side of the printer to maintain the cords from getting tangled in the heating elements. I also added a Raspberry Pi with a camera that is running OctoPi. In the WebGUI of OctoPi I can control almost all the features of the printer plus with the added camera I can view my prints when I'm away from the printer. 

## Desk and Tools

I also have a desk near by the 3D printer where I have tons of drawers all with all seperated electronic parts that I either disassembled or bought in bulk online. When I have an idea I try to sketch it out and when I get back to my lab try to assemble it or draw out a PCB to test my idea out. There are a few cool little projects that I have made and would love to share more about those in later post.