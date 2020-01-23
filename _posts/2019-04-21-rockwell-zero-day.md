---
layout: post
title: "Zero-Day Research:  Rockwell Automation MicroLogix 1400 and CompactLogix 5370 Controllers"
featured-img: rockwell0day
author: Josiah Bryan & Geancarlo Palavicini
category: [Zero Day Research]
---

## Background

As technology continues to advance and more devices become networked together, new vulnerabilities will inevitably rise to the surface that security teams will have to deal with. The critical infrastructure sector is no exception to this phenomenon, where common web technologies are being found more frequently in [Programmable Logic Controllers](https://en.wikipedia.org/wiki/Programmable_logic_controller) (PLCs), a keystone piece of any industrial network. 
During our SCADA (Supervisory Control and Data Acquisition)/ICS (Industrial Control System) research over the years, we have seen this trend firsthand, which lead us to examine whether the introduction of  web technologies would also introduce many of the vulnerabilities that have plagued websites for decades.  This post will cover an Open Redirect Zero Day vulnerability we discovered in Rockwell Automation controllers and provide a POC python exploit to help demonstrate our findings. 

### DISCLAIMER
This post is for education purposes ONLY.  Exploiting a live PLC is illegal.  We do not condone illegal activity and cannot be held responsible for any misuse of the given information.

## AFFECTED PRODUCTS

The following Rockwell Automation products are affected: 
* MicroLogix 1400 Controllers
* Series A, All Versions
* Series B, v15.002 and earlier
* MicroLogix 1100 Controllers v14.00 and earlier
* CompactLogix 5370 L1 controllers v30.014 and earlier
* CompactLogix 5370 L2 controllers v30.014 and earlier
* CompactLogix 5370 L3 controllers (includes CompactLogix GuardLogix controllers) v30.014 and earlier

## Open Redirect Vulnerability
An Open Redirect vulnerability occurs when a web application accepts user-supplied input in the URL that contains a link to an external website, and consequently uses that link to redirect the user's browser, providing a mechanism for attackers to install malicious software on the user's machine. The Open Redirect vulnerability we discovered in various Rockwell Controllers is no exception to this rule.  Each controller runs a web server that displays diagnostics about that specific PLC.  There also exists a URL parameter that enables the PLC to redirect the user's browser like so:
```
http://192.168.1.12/index.html?redirect=/localpage
```
The redirect parameter intends to send users to another page located on the PLCs website.  Under normal circumstances, the PLC would filter out redirects to an external website, so if you tried the following:
```
http://192.168.1.12/index.html?redirect=/externalsite
```
it would filter out the request and prevent the browser from being sent to a malicious site.  However, the PLCs redirect filter does not account for the various ways to enter a URL like so:
``` 
http://192.168.1.12/index.html?redirect=//MaliciousSite.com
```
From the browser's perspective, the second '/' character will be ignored, making it a valid URL yet bypassing the PLCs filter.  The browser will then be redirected to the website provided after the second '/'.  This type of client-side attack can aid in phishing campaigns to setup browser exploits or install malware. 

## Proof of Concept Exploit

Consider the following scenario:

A user with access to the controller receives an email disguised as the Support Center or Help Desk (Figure Below):

![HelpDesk](/assets/img/posts/helpdesk.png)

Upon following the link, you can see the user is prompted to save the executable being served on the external site:

![RedirectMalware](/assets/img/posts/redirectmalware.png)

This basic example demonstrates how the browser is redirected to an external page, setting up the stage for far more complex browser attacks.  This attack could indirectly pose a real threat to the control system if the attacker manages to get direct access to a machine that has access to the controller.  The following POC code will generate a redirect link for you to replicate the attack in a controlled environment:

``` python
import argparse

parser = argparse.ArgumentParser(description='Callback Script')
parser.add_argument('-r', '--redirect', required=True, dest="redirect", action='store', help='Redirect Destination IP')		
parser.add_argument('-p', '--plc', required=True, dest="plc", action='store', help='Rockwell Controller IP')	
args = parser.parse_args()  #Parse Command Line Arguments

print "Generating link..."
print "http://"+args.plc+"/index.html?redirect=//"+args.redirect

```
## Disclosure

The vulnerabilities were immediately reported to the National Cybersecurity and Communications Integration Center (NCCIC) by security researchers Josiah Bryan and Geancarlo Palavicini.  You can find the full advisory [here](https://ics-cert.us-cert.gov/advisories/ICSA-19-113-01).

## Mitigation
Rockwell Automation has released an update for each of the affected devices. Rockwell also recommends users take defensive measures to minimize the risk of exploitation of this vulnerability. Specifically, users should:
* Update to the latest available firmware revision that addresses the associated risk.
* Use trusted software, software patches, anti-virus/anti-malware programs, and interact only with trusted
websites and attachments.
* Minimize network exposure for all control system devices and/or systems, and ensure that they are not accessible from the Internet.
* Locate control system networks and devices behind firewalls and isolate them from the business network.
* When remote access is required, use secure methods such as virtual private networks (VPNs), recognizing that VPNs may have vulnerabilities and should be updated to the most current version available. VPN is only as secure as the connected devices.
* Employ training and awareness programs to educate users on the warning signs of a phishing or social engineering attack.

