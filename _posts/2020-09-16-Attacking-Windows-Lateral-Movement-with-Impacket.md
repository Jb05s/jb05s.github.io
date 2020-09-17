---
title: "[UNDER CONSTRUCTION] Attacking Windows: Performing Lateral Movement with Impacket"
date: 2020-09-16
tags: [posts]
excerpt: "What's the differences between WMIExec, PSExec, and SMBExec? Let's take a closer look at each of these tools and get a better understanding of what's happening when we execute them against a target."
---
Overview
---
In this short post, I'd like to discuss a few tools available in [Impacket](https://github.com/SecureAuthCorp/impacket) and what these tools are doing "under the hood".

__Note: The processes shown in this blog post were performed in a personal lab environment and for the sole purpose of discussing the topics covered in this post.__

The Situation
---
So let's say you're on an internal network assessment. You've established an initial foothold through various means and began performing network enumeration. For simplicity in the blog post, to gain a foothold on the network, I'll be using [Responder](https://github.com/SpiderLabs/Responder) to capture and crack an NTLMv2 hash offline using [Hashcat](https://github.com/hashcat).

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/responder.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/hashcat.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/begin-enum.png" alt="">

Through your enumeration, you've identified that the user account initially compromised has Administrative privileges on machines in the network. 

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/powerview-clientA-admin.png" alt="">

From here, you'll want to perform lateral movement to gain access to those machines. This blog post will cover some of these lateral movement techniques. Let's talk about some ways we can go about performing lateral movement - specifically with PSExec, SMBExec, and WMIExec available with Impacket.

PSExec
---
- PSExec sends/receives data over SMB (Port 445) via a named pipe
	- Drops files to disk to startup a service and enable a named pipe (Can be detected by AV)
	- Loudest of the three techniques


SMBExec
---
- SMBExec is very similar to PSExec, however, does not drop a binary to disk
	- Echoes and executes a batch file containing the command string to execute
	- Saves the command output to a temp file
	- Every command executed in SMBExec is run in a new service
	- Stealthier than PSExec, but still leaves quite a lot of logs


WMIExec
---
Port(s) Utilized - Sends input over RPC/DCOM (TCP Port 135 --> TCP Port >= 1024) and receives output over SMB (TCP Port 445)

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-cmd.png" alt="">

- Writes temporary file to the ADMIN$ directory
	- Filename is a date to timestamp coversion from the initial time of connection

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/temp-file-wmiexec.png" alt="">

- Known to be used by administrators (Stealthiest option)

Wrapping Up
---
<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/mimikatz.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-pth.png" alt="">