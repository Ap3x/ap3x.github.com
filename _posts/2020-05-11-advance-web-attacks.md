---
title:  "Advance Web Attacks"
date: 2020-05-16 16:47:17 -0400
categories: [Offensive Security, OSCE]
tags: [osce, exploit, development, web, security, xss, lfi, csrf]
---

## Introduction

This content is gathered from multiple different resources that I have added all to this post that helped me understand and learn more about exploit development. There are a lot of good articles, whitepapers, and practice that help teach different types of Advance Web Attacks.

#### Types Of XSS

The most common XSS vulnerablities are stored cross site scritping and reflective cross site scripting.

- **Stored XSS**
    - When the application recieves the data from the attacker and includes it later in HTTP responses in an unsafe way.
        -  **Example:** Blog post or comment section that allows the user or attacker to enter text in a field and save it to the page. This can be an issue to the developer and the clients that visit the page because each time client or victim visits the page the script or code that we stored into the page is executed since the page is being loaded. This opens the possibility for the attacker to collect cookies, usernames, and other info that may be used for accessing the victims data
        - **Testing:** In order to find XSS attack vectors you neex to test all the relevant entry and exit points via attacker-controllable data. The entry points may include parameters that are passed or other data that is passed in the URL string. or the URL file path. The exit points may be found in HTTP responses that are returned from any application. It may be helpful to use something like BurpSuite to automate testing for XSS
        - **Links:** 
            - https://portswigger.net/web-security/cross-site-scripting/stored
            - https://www.exploit-db.com/papers/13646
- **Reflected XSS**
    - A reflected XSS attack can arises when an application recieves data in an HTTP request and and includes the same data in the reponse in an unsafe way.
        - **Example:** If a website has a search function which recieves the user suppliced search term in a URL parameter. Then the application echos that same data that you searched in the response page. 
        - **Testing:** Since a Reflective XSS attack isnt as severe as a stored XSS this attack is usually directed at a single user or a specific set of victims. The attacker my supply the user a URL of the site that has a reflective XSS vuln. and supply that to the victim to ge the victim to click the link and echo the script or code onto their webpage. The impact of XSS attackes allows the user to modify, view, or sometimes delete anything the victim may alos have access to. This allows the attacker to control the users session.
        - **Links:** 
            - https://portswigger.net/web-security/cross-site-scripting/reflected
            - https://www.exploit-db.com/papers/13646
- **DOM XSS**
    - DOM-based XSS are a rare attack method and isn't as commonly seen. DOM stands for Document Object Model. A DOM-based XSS arises when an application contains some client-side Javascript that processes data from an untrusted source in an unsafe way that usually ends up writing back to the DOM.
        - **Example:** If an application uses some javascript to read the value of an input filed and write that value to an element within the HTML. Then the attacker can control the value of the input field and can cause there malicious code to execute.
        - **Links:**
            - https://portswigger.net/web-security/cross-site-scripting/dom-based

#### LFI and RFI
There are two different types of File Inclusion attack methods one is local and the other is remote. In regards to LFI the attacker is able to retrive a file that is locally stored on the victims machine which is usually a web server hosting a website. The other type of file inclusion attack method is RFI and this attack method is triggered when an the web server is retriving external files from the victim machine.
- **Local File Inclusion (LFI):**
    - A piece of vulnerable code would look something similar to what is seen below.
    ```php
    <?php
	$template =$_GET['template'];
	include("/".$template .".php"); //<-- Vulnerable !!
	?>
    ```
    As you can see the code above the code is including a php file from the current directory. Usually on linux web servers the web directory is in /var/www/ folder. If the attacker was targeting a linux web server then we can try to retrieve files from the system such as /etc/shadow which contains the users and their respective password hash. In order to retrieve local files the way we would retrieve the file would be similar to traversing the file system or changing directories like so "cat ../../../../etc/shadow". This linux command will output the shadow to the webserver page. So the attacker would uses something similar to:
    > http://vulnsite.com/index.php?template=../../../../../etc/shadow

    A %00 is usually added to the end of the URL to tell the web server that this is a NULL character and everything after the %00 is voided or ignored.

- **Remote File Inclusion (RFI):**
    - A piece of vulnerable code would look something similar to what is seen below.
    ```php
    <?php
	$file =$_GET['page'];	//The page we wish to display
	include($file .".php");	// <-- Vulnerable !!
	?>
    ```
    As you can see the code doesn't perform any sort of checks on the content of the 'page' parameter which will allow the attacker to put their file or PHP reverse shell into the webpage. The attacker would exploit this RFI by passing a URL that is a link to the PHP reverse shell as the parameter on 'page' in this example.

- **LFI -> RCE**

These sorts of vulnerabilities occur in PHP functions where developers don't check the user supplied data

RFI/LFI PHP functions:
- include()
- include_once()
- require()
- require_once()
- fopen()

There are several way to go from a LFI/RFI to Remote Code Execution (RCE). One of the ways to get RCE is by Poisoning the Apache Logs. In the log it keeps track of the GET and POST requests that are made to the server, but if the attacker were to change URL header that they are requesting our logs will reflect that in the log file. We can then leverage the log file with our poisoned code by using a LFI this we display the log file that has the malicious code as seen below. 

Another method to to get remote code execution from an LFI is through process environment. When a user visits a PHP page on the web server a process is created in the *nix system. Each process has its own entry that can be found in the /proc/self/ directory that creates a static path and a symbolic link from the latest process used that contains useful information. If we can inject some sort of malicious code in the /proc/self/environ, we can run arbitrary command from target through the LFI we have found. If you notice in a /proc/self/environ instance it contains info about the User-Agent. We can leverage this and in order to poison an instance the attacker can use a tool like BurpSuite to alter the User-Agent value to our malicious code to something similar to below. Then using the LFI the attacker can navigate to that file and use the CMD parameter to get code execution.

Another way to go from LFI to RCE is by assuming that we have access to uploading a file to the system. If the attacker was able to upload a malicious file such as a simple php line like seen below then you could point the LFI to that file and use the cmd parameter to execute a command on the system.

```php
<?php exec($_GET[cmd]);>
```

### Cross Site Request Forgery (CSRF)

A successful cross site request forgery attack carried out by an attacker will cause the victim user to execute an unintentional action on a web browser. A good example of a successful CSRF attack would trick the user to transfer funds from their bank account to another bank account and the attacker. In summary the attacker can gain access or full control of all the web application data and functionality of a victims account through a forged request using browser cookies.

So how does a CSRF work? In its simplest form the way a CSRF works is first the attacker has to identify a relevant action such as privileged action such as changing a username, deleting an account, or transferring money. Once the attacker has identified a action to leverage then the attacker can check if the application relies solely on the session cookies to identify the user who is authorizing the action. If there is no other method the application takes to verify that the user made the action then the attacker can begin to forge a request. By default browsers trust the code that is given to it and will send the session cookies the site if it is believed to be trusted.

There are many ways to target a user using a CSRF. For example an attacker can add some malicious code to a fake site that could have a hidden iframe or just built into the site page. The malicious code would make a POST request to the web application that is vulnerable to the CSRF attack. So to piece it all together when the victim navigates to fake site with the attackers malicious code the victims browser will be tricked to handing the session cookie of our target application to the malicious code and bundle all it into a POST request and send that to the target web application. The POST request would be the privileged action that the attacker takes advantage of to gain access or modify the application data.

**Links**
- https://www.youtube.com/watch?v=vRBihr41JTo
- https://portswigger.net/web-security/csrf

