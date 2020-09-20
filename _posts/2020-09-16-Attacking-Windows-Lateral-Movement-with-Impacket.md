---
title: "Attacking Windows: Performing Lateral Movement with Impacket"
date: 2020-09-19
tags: [posts]
excerpt: "What's the differences between WMIExec, PSExec, and SMBExec? Let's take a closer look at each of these tools and get a better understanding of what's happening when we execute them against a target."
---
Overview
---
In this short post, I'd like to discuss a few tools available in [Impacket](https://github.com/SecureAuthCorp/impacket) and what these tools are doing "under the hood".

__Note: The processes shown in this blog post were performed in a personal lab environment and for the sole purpose of discussing the tools covered in this post.__

The Situation
---
So let's say you're on an internal network assessment. You've established an initial foothold and obtained user credentials, and began performing network enumeration. For simplicity in the blog post, for obtaining valid user credentials, I'll be using [Responder](https://github.com/SpiderLabs/Responder) to capture and crack an NTLMv2 hash offline using [Hashcat](https://github.com/hashcat).

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/responder.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/hashcat.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/begin-enum.png" alt="">

Through your enumeration, you've identified that the user account you obtained has Administrative privileges on a machine in the network. 

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/find-localadminaccess-usera.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/powerview-clientA-admin.png" alt="">

From here, this is where you'll want to perform lateral movement to gain access the machine. This blog post will cover some lateral movement techniques that can be used. 

Let's talk about some ways we can go about performing lateral movement - specifically with PSExec, SMBExec, and WMIExec available in Impacket.

PSExec
---
This first Impacket tool we'll discuss is PSExec. PSExec allows users to connect to remote machines and execute commands over a named pipe. The named pipe is established through a randomly-named binary file that's written to the ADMIN$ share on the remote machine and utilized by SVCManager to create a new service. 

You can imagine this step as running: `sc create [serviceName] binPath= "C:\Windows\[uploaded-binary].exe"`.

Once the named pipe is established, all command input and output between you and the remote machine is communicating over the SMB protocol (445/TCP).

Let's take a closer look at what PSExec is doing.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/psexec-diagram.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/psexec-cmd.png" alt="">

As shown in the image above, we're writing the binary file `PDSwlQrA.exe` to the ADMIN$ share on the remote machine. We can verify if the binary file exists on the remote system by listing the files and directories in `\\127.0.0.1\ADMIN$`.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/psexec-binary.png" alt="">

Now that the binary has been successfully written to the remote machine, this is where SVCManager steps in to start a service and create that named pipe back to our machine. (We'll see later on, when reviewing the remote machine's event logs, that SVCManager was called.)

With a general idea of what the tool is doing, what are the potential artifacts that could be left behind.. and what event logs are we generating on the remote system?

Let's say we're utilizing PSExec and our connection abruptly errors out or you accidently close the process window running PSExec. This means that artifacts on the remote system weren't properly cleaned up and we need to go back to manually clean them up ourselves. (Mind you, this applies to nearly all situations where an abrupt error that causes the connection to die).

In regard to PSExec, you will have to make your way back onto the remote machine and remove the binary file, kill the service, and verify that whatever you were performing at the time of the abrupt error is killed.

As for event logs generated on the remote machine, here's a breakdown:
- Event logs generated (To establish communication and run a single command, then exit PSExec)
	- 1 System Event IDs: 7045 (Service Started)
	- 12 Security Event IDs: 4672 (Special Privilege Logon), 4624 (Logon), 4634 (Logoff)

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/psexec-eventlog-sys.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/psexec-eventlog-sec.png" alt="">

In the next section, we'll be talking a little bit about SMBExec. SMBExec operates quite similarly to PSExec, but has some minor differences.

SMBExec
---
As just mentioned above, SMBExec is very similar to PSExec, however, SMBExec doesn't drop a binary file to disk. SMBExec utilized a batch file, along with a temporary file, to execute and relay messages back. Just like PSExec, SMBExec sends input and receives output over the SMB protocol (445/TCP).

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-diagram.png" alt="">

Let's take a closer look at how SMBExec works. Let's use SMBExec to establish an interactive connection to the remote machine.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-cmd.png" alt="">

As we can see in the image above, we've successfully established a connection to the target machine. For analysis purposes, let's execute a command to request an instance of `Notepad.exe` to run.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-notepad-run.png" alt="">

As we'll quickly see, we lose our ability to send further input to the remote machine. This is happening because we're still waiting for the command output from the remote machine.. which we'll never receive. This situation is ideal for analysis over on the remote machine.

If we head over to the remote machine and open up an instance of [Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer), we can locate the Notepad.exe process and review the process tree.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-procexplorer.png" alt="">

We can see that the Notepad.exe process is a child process to CMD.exe. If we hover over CMD.exe, we can see that it's executing whatever data is stored in `C:\Windows\TEMP\execute.bat`. Let's quickly read what data is stored in this file.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-notepad.png" alt="">

Upon reading the data in the `execute.bat` file, we notice that our input sent over to the remote machine was appended to the beginning of the file.

The batch file is essentially taking our input sent over to the remote machine, executing it, and redirecting the output to a temporary file named `__output` located in `\\127.0.0.1\C$`.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-output.png" alt="">

For event logs generated by SMBExec on the remote machine, here's a breakdown:
- Event logs generated (To establish communication and run a single command, then exit SMBExec)
	- 4 System Event IDs: 7045 (Service Started), 7009 (Service Error - Timeout)
	- 3 Security Event IDs: 4672 (Special Privilege Logon), 4624 (Logon), 4634 (Logoff)

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-eventlogs.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/smbexec-eventlogs-sec.png" alt="">

The last tool talked about in this post will be WMIExec. WMIExec, when compared to PSExec and SMBExec, operates quite differently. Let's talk about it now.

WMIExec
---
WMIExec (Windows Management Instrumentation) allows remote access to machines by initially communicating with Remote Procedure Calls (RPC) over TCP port 135. After initial communication is established, a random port over 1024 is used for negotiation. 

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-diagram.png" alt="">

This connection is used to send input to the remote machine. The input is executed in a CMD.EXE process and the output is saved to a temporary file within the ADMIN$ share on the remote machine. The temporary file can be easily be identified in the ADMIN$ share by looking for a filename starting with "`__`". The numerical value proceeding the two underscores may seem random, however, it's actually the current date and time converted to a timestamp.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-cmd.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/temp-file-wmiexec.png" alt="">

Let's take a closer look at what's going on here. Again, for the sake of analysis, let's request the remote machine to start an instance of Notepad.exe. We'll notice that by executing this command will cause WMIExec to hang.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-notepad.png" alt="">

If we jump over to the remote machine and fire up Process Explorer, we can identify and analyze Notepad.exe.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/procexplorer-wmiexec.png" alt="">

As we can see, highlighted in purple, the input we sent is being executed within a CMD.exe process. We can also notice that the output from the command is being redirected to the temporary file located in the ADMIN$ share.

After the threads within the process complete their tasks, the process is terminated and the output is written to the temporary file. Then the output stored within the temporary file will be sent back to our machine over SMB.

__Note: It's worth noting that if you abort WMIExec while waiting for output back from the remote machine the temporary file will not be deleted on the remote system.__

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-eventlogs.png" alt="">

- Event logs generated (To establish communication and run a single command, then exit WMIExec)
	- 14 Security Event IDs: 4672 (Special Privilege Logon), 4624 (Logon), 4634 (Logoff)

Wrapping Up
---
Now that we've moved laterally on the network to a machine we have Administrative privileges, we can now perform additional enumeration to identify our next steps to gaining privileged access on the domain network.

One option could be dumping the memory of the Local Security Authority Subsystem Service (LSASS.EXE) process and extract/view cached credentials (Cleartext passwords, NTLM hashes, etc.).

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/mimikatz.png" alt="">

What if you run into a situation where you've successfully extracted credentials from the LSASS.exe process, but you only have an NTLM hash of a user with admin privileges somewhere else on the network?

Impacket (and other tools) also supports pass-the-hash techniques. In the following screenshot, we're able to supply the NTLM hash of a User in Impacket's - WMIExec, as an argument, to obtain an interactive shell on a remote machine that we have admin privileges.

<img src="{{ site.url }}{{ site.baseurl }}/images/attacking-windows-impacket/wmiexec-pth.png" alt="">

In my next post, I'll talk about the pass-the-hash technique, along with how NTLM authentication works. Stay tuned and stay safe!

