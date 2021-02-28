---
title: HTB Dante Pro Lab and THM Throwback AD Lab
date: 2020-11-16 15:42:43 -0400
categories: [Pentesting , Lab]
tags: [HTB, HacktheBox, Tryhackme, THM, pentesting, AD]
---


# Summary
Over the course of a couple months I've been really busy with school and trying to finish my undergraduate degree in Computer Science and Engineering, but I managed to squeeze in some time between family and school to try out two different labs that I've been hearing a lot about. In this post I gonna give a my opinion and thoughts about the lab and not reveal any solutions.

## Hack The Box Dante Pro Lab
This lab is by far my favorite lab between the two discussed here in this post. This lab took me around a week to complete with no interruptions, but with school and job interviews I was slowed down a bit more and took a little longer than expected. The lab is great for someone that maybe preparing for their OSCP or maybe for someone that freshly completed their OSCP and wants another challenge. Just a heads up this lab did have a couple very basic buffer overflows that you had to overcome. The lab also has Active Directory aspect in it, but does not consist of anything that you need years of experience to break and is fairly simple. The only thing I would consider is to know your tooling and when/where to use the proper tools. The best part about the lab in my opinion was learning how to laterally move through the network. I personally used [chisel](https://github.com/jpillora/chisel) to help me pivot. Also if you haven't already join the Hack the Box discord if you need help with anything usually people are always willing to help. In all I had a lot of fun. Met some cool people along the way and earned this cool certification of completion.

### Certificate of Completion
<img src="/assets/img/post/2020-11-16-htb-dante-pro-lab-and-thm-throwback-network-lab/Dante-Lab.png" style="width: 80%" >


## Throwback 
This network is for those who are complete beginners when it comes to Active Directory which is great because this fit perfectly with the other content on TryHackMe. When you sign up for the lab you can either go through the lab as if each machine are "Black Boxes" or you can follow along with prompts and hack the network in order. When I took this lab I completed it before some of the more famous Youtubers did walk-through of the network, but now that those videos are out there you can watch them freely for help if you stumble. Also this lab had some glitches in version 1.0 of the Throwback network when I was going through the network. Although I was told that they will improve this in the future. There were also some tools of my own that I just couldn't run properly. For a day or so I contemplating if it was my tools that were the issue or not. I ended up testing my tools on some Windows vms I had to ensure I wasn't going crazy and everything seemed to work on my end. Other than the few glitches I encountered I learned a lot and how about navigate to me at the time is a large network.

<img src="/assets/img/post/2020-11-16-htb-dante-pro-lab-and-thm-throwback-network-lab/Throwback2.png" style="width: 80%" >

 The only real experience I've had was building an AD environment in my homelab with a couple computers to test a few tools and networking reasons. I've also replicated the CCDC network environment for our Cyber Defense team, but not to the extent of dealing with AD forest. I would definitely recommend this lab to someone who has never had experience with Active Directory and needs some guidance. This lab was easy, but there were only one or a couple ways to exploit the machines which forces the student to use the intended methods outlined. I hope in future labs they add multiple ways to break in the machines. This would make me want to stay in the labs even longer, but I think someone can easily complete this lab in a week or two as a beginner and still absorb all the info taught.

### Certificate of Completion

<img src="/assets/img/post/2020-11-16-htb-dante-pro-lab-and-thm-throwback-network-lab/Throwback.png" style="width: 80%" >

