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

Through your enumeration, you've identified that the user account initially compromised has Administrative privileges on a machine in the network. 

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/find-localadminaccess-usera.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/powerview-clientA-admin.png" alt="">

From here, you'll want to perform lateral movement to gain access the machine. This blog post will cover some of these lateral movement techniques. Let's talk about some ways we can go about performing lateral movement - specifically with PSExec, SMBExec, and WMIExec available with Impacket.

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
WMIExec (Windows Management Instrumentation) allows remote access to machines by initially communicating with Remote Procedure Calls (RPC) over TCP port 135. After initial communication is established, a random port over 1024 is used for negotiation. 

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-diagram.png" alt="">

This connection is used to send input to the remote machine. The input is executed in a CMD.EXE process and the output is saved to a temporary file within the ADMIN$ share on the remote machine. The temporary file can be easily be identified in the ADMIN$ share by looking for a filename starting with "`__`". The numerical value proceeding the two underscores may seem random, however, it's actually the current date and time converted to a timestamp.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-cmd.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/temp-file-wmiexec.png" alt="">

Let's take a closer look at what's going on here. For the sake of analysis, let's request the remote machine to start an instance of Notepad.exe. We'll notice that by executing this command will cause WMIExec to hang.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-notepad.png" alt="">

If we jump over to the remote machine and fire up [Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer), we can identify and analyze Notepad.exe.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/procexplorer-wmiexec.png" alt="">

As we can see, highlighted in purple, the input we sent is being executed within a CMD.exe process. We can also notice that the output from the command is being redirected to the temporary file located in the ADMIN$ share.

After the threads within the process complete their tasks, the process is terminated and the output is written to the temporary file. Then the output stored within the temporary file will be sent back to our machine over SMB.

__Note: It's worth noting that if you abort WMIExec while waiting for output back from the remote machine the temporary file will not be deleted on the remote system.__

Wrapping Up
---
Now that we've moved laterally on the network to a machine we have Administrative privileges, we can now perform additional enumeration to identify our next steps to gaining privileged access on the domain network.

One option could be dumping the memory of the Local Security Authority Subsystem Service (LSASS.EXE) process and extract/view cached credentials (Cleartext passwords, NTLM hashes, etc.).

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/mimikatz.png" alt="">

What if you run into a situation where you've successfully extracted credentials from the LSASS.exe process, but you only have an NTLM hash of a user with admin privileges somewhere else on the network?

Impacket (and other tools) also supports pass-the-hash techniques. In the following screenshot, we're able to supply the NTLM hash of a User in Impacket's - WMIExec, as an argument, to obtain an interactive shell on a remote machine that we have admin privileges.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-pth.png" alt="">

In my next post, I'll talk about the pass-the-hash technique, along with how NTLM authentication works. Stay tuned and stay safe!

