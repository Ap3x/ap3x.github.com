---
title: "How to Install Ghidra on Ubuntu 19.10"
date: 2020-04-17 09:00:00 -0400
categories: [How To, Install Ghidra on Ubuntu 19.10]
tags: [ghidra, reverse engineering, how to]
---

In order to run Ghidra you need to install the Java Runtime Environment

First Update
> sudo apt update

Next install the OpenJDK:
> sudo apt install default-jre

Verify that you have install java
> java -version

Next Install OpenJDK 11 JDK and the OpenJDK 11 JRE
> sudo apt install openjdk-11-jdk 
> sudo apt install openjdk-11-jre

Now Navigate to `https://ghidra-sre.org/` and click the Download.

Once you have downloaded Ghidra it should be in your downloads folder unless you changed it to another folder. Then go into that folder and extract it.

> sudo unzip ghidra_9.1.2_PUBLIC_20200212.zip

Once you have unzipped the folder go ahead and go into that folder:

> cd ghidra_9.1.2_PUBLIC

Then run the executable:

> ./ghirdaRun









