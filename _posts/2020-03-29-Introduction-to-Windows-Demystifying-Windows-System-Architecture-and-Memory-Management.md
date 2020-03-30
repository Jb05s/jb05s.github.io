---
title: "Introduction to Windows: Demystifying Windows System Architecture and Memory Management"
date: 2020-03-29
tags: [posts]
excerpt: "How Does Windows Work?! Let's talk about Windows Memory Management and General Architecture"
---
Overview
---
In this post, I'd like to give some insight on a few topics that I've struggled wrapping my head around, or just tend to find myself running back to my notes for clarification.

Here are the topics I will touch upon in this post:
1. Kernel Mode vs. User Mode
2. Processes and Threads
3. Objects and Handles
4. Virtual Memory
5. Function Call Flow

While all of these topics could have their own individual blog post, this blog post is meant to be an introduction. I may release posts in the future that go a little more in-depth.

I'm going to walk through the process of what happens when you, as a user, perform everyday tasks on a Windows operating system. This walkthrough will take you "under the hood" of what goes on in Windows.

Let's jump right into it!

Kernel Mode vs User Mode
---
To begin, in Windows, to protect user applications from accessing critical operating system data, Windows splits the processor up into two access modes. These two modes are User mode and Kernel Mode.  

This schema ensures that any user application that's performing unintended actions won't disrupt the stability/avilability of the overall system.  

<img src="{{ site.url }}{{ site.baseurl }}/images/privilege-rings.png" alt="">

In the illustration shown above, there's a total of seven layers (or rings) of protection.

Whenever code executes, it always will have a mode associated with it. This is called the "Thread Access Mode" or better known as the "Current Privilege Level (CPL)"

While there may be several rings, Windows only uses two of these rings:
1. Ring 0 (Kernel Mode); which is the most privileged
2. Ring 3 (User Mode); which is the least privileged

Here's some key differences between User Mode and Kernel Mode:
1. User Mode (CPL 3)
	- Cannot directly access hardware
	- Cannot access some parts of Virtual Memory
	- Need to call System Calls to perform some operations
	- Has restricted access
2. Kernel Mode (CPL 0)
	- Can directly access hardware
	- Can access any part of Virtual Memory
	- Doesn't need to call System Calls (But has the ability)
	- Has unrestricted access

User Mode (CPL 3)
---
User Mode is responsible for running code within user applications.

This mode doesn't allow access to operating system code or data, and is denied access to system hardware.  
If a crash occurs in this mode, the system is not effected, only in the application where the error occurred.

Kernel Mode (Privileged) (CPL 0)
---
In this mode, it has complete access to the kernel and device drivers.  

Additionally, this mode is allowed to access all system resources.  

Any unhandled exception in kernel mode can result in a system crash, infamously known as the Blue Screen of Death (BSoD).

<img src="{{ site.url }}{{ site.baseurl }}/images/bsod.png" alt="">

Based on the CPL you're operating in, you'll have the ability to read and write data in the segments of that CPL and of that of the lesser.

In order to go from User Mode (CPL 3) to Kernel Mode (CPL 0), a "Call Gate" needs to be called. I'll shine some more light on call gates later in the post, when I talk about the flow of a function call.

We can get a bit of a visual by viewing the 'Performance' tab within Task Manager. See the figure below for detail.

<img src="{{ site.url }}{{ site.baseurl }}/images/task-manager-performance.png" alt="">

As seen in the visual, we can see when the CPU is performing operations in User Mode vs Kernel Mode. You're able to see this graph layout by right-clicking in the graph and selecting `'Show kernel times'`.

Processes
---
Processes are management containers for threads to execute code.  
There's multiple occasions where I've heard "There's a process running..". This is an inaccurate statement. A process is never "running".  
A process is simply a container (or manager) for providing resources to execute calls. Threads are what's actually "running" or executing code, not processes.  
A process consists of the following:
- A private virtual address space
- An executable program containing data that can be executed
- A table of handles referencing kernel objects
- An access token; Used for security checks when accessing shared resources
- Manage one or more threads; the entity that performs code execution

We can view the list of current processes in _'Task Manager'_ or _'Process Explorer'_, as seen in the figure below.

<img src="{{ site.url }}{{ site.baseurl }}/images/processes.png" alt="">

<img src="{{ site.url }}{{ site.baseurl }}/images/processes-procexplorer.png" alt="">

Process Creation
---

(Walk through th e data structure in WinDbg (ie. EPROCESS, KPROCESS, KTHREADS, PEB, etc.))

Threads
---
Threads, as mentioned earlier, are entities scheduled by the kernel to execute code.  
A thread consists of the following:
- The state of the CPU registers
- Current Access Mode (User or Kernel)
- User and Kernel stacks
- A private storage area, called Thread Local Storage (TLS)
- Optional security token (Can perform impersonation to run as a higher/lower privilege level)
- Optional message queue and Windows the thread creates (Running or Not Responding)
- A priority (0 (Lowest)-31 (Highest))
- A state: Running, Ready, Waiting

These are the owners of a window (i.e. Notepad) that receive user-input.  
So, when you open Task Manager and see "Status - Running" or "Status - Not Responding", this is simply stating if a thread is ready to receive input, or that a thread is working on an operation.  
When you're presented with the "Status - Not Responding", this is saying that the thread has been held up for five or more seconds while trying to complete an operation.

To present a visual on threads, let's take a look at two instances of Notepad.exe.  

When a user runs an application; such as Notepad.exe, an initial thread is created.

<img src="{{ site.url }}{{ site.baseurl }}/images/threads-init-notepad.png" alt="">

In the first instance of Notepad, it will be waiting for the user to request some kind of operation to be performed. Since it's not performing any kind of operation, there should only be one thread assoicated to this process.

Alternatively, in the second instance of Notepad, we're going to perform an operation. For this example, let's try and open a file in Notepad. Upon clicking into the `File` tab in Notepad, and clicking the `Open File` option, we'll be able to see that additional threads are being created to carry out the request. See the visual below for details.

<img src="{{ site.url }}{{ site.baseurl }}/images/notepad-compare.png" alt="">

If you'd like to see each individual thread, the thread identifier (TID), etc., we can use _'Process Explorer'_.

<img src="{{ site.url }}{{ site.baseurl }}/images/thread-procexplorer.png" alt="">

Now that we've got some insight on processes and threads, let's proceed by discussing what objects and handles are, and how they come into play.

Objects and Handles
---
An object is a static data structure that reside in system memory space and represent runtime resources like processes, threads, etc., and provides information on various attributes (i.e. object type).  

All objects are managed by the Object Manager. The Object Manager is responsible for creating, managing, and terminating objects.  

Kernel code can obtain a direct pointer to an object. In order for user mode code to get access to an object is by using a handle.  

A handle is an index in a table that points to a specifc object in kernel space.  

They're used as shields against user code from directly accessing an object in the kernel.
- Handle values are always in multiples of 4  
- Objects are reference counted
- The Object Manager is responsible for managing the reference counter
- Every process has it's own Handle Table

(Process Explorer Visuals - See Windows Internals 'Demo: Objects and Handles')

Virtual Memory
---
Windows implements virtual memory based on a linear memory model. Every process has it's own private virtual address space.  
The virtual address space for a process is divided into two parts; a User Space and a Kernel Space.

Let's use the following diagram to go into more detail.

<img src="{{ site.url }}{{ site.baseurl }}/images/virtual-memory-mapping.png" alt="">

Virtual memory may be mapped to physical memory, but can also be stored on disk (Page file)
Processes access memory (via a pointer) regardless of where it resides
- The memory manager handles the mapping of virtual to physical pages
- Processes don't know the actual physical address of a given virtual memory address
	- Processes always use the virtual memory address
Virtual Memory Mapping:
RAM (Physical Memory) is shared by all processes
When a process allocates memory, it is mapped to physcial memory (Blue blocks in diagram)
- These chunks of memory are known as pages
	- Pages are 4kb in size
	- Chunks of memory, or pages, don't need to be contiguous in physical memory, even if they're contiguous in virtual memory
- In some occasions, virtual memory may be mapped temporarily to disk, if the physical memory is being used up by other processes
- Sometimes, virtual memory from different processes can be mapped to the same physical page (meaning the page is shared)
	- For example; two processes resourcing for 'Notepad.exe'
		- Windows will try and share resources as much as possible, if it makes sense to.

In task manager or process explorer, under the 'Details' tab, you can see the 'Memory (Private Working Set)' column.

<img src="{{ site.url }}{{ site.baseurl }}/images/private-working-set.png" alt="">

- 'Private' meaning the memory is not shared
	- Allocating memory via 'malloc' is considered private memory, this is what is shown
	- Loading a DLL is sharable memory, and isn't shown in this column but in the 'Memory (Shared Working Set)' column
		- To get a better idea of how much memory a process consumes, view the 'Commit Size' column
- 'Working Set' is a term used to describe physical memory
	- This column shows how much memory is being used by the process
- The values under this column are always divisible by 4 (The size of a memory page)

<img src="{{ site.url }}{{ site.baseurl }}/images/commit-memory-size.png" alt="">

Virtual Memory Layout
---
Now, let's talk a little bit about Virtual Memory, how it works, and the differences between 32-bit and 64-bit virtual memory space.

<img src="{{ site.url }}{{ site.baseurl }}/images/virtual-memory-layout.png" alt="">

As seen the in 'Virtual Memory Layout' illustration, shown above, 32-bit and 64-bit architecture have quite a huge gap between one another.

On a 32-bit architecture, we can see that it has a total of 4 GB of virtual memory.  
The memory is split into User Process Space (Lower Addresses) and System Space (Higher Addresses).  
- 2 GB for User Process Space
- 2 GB for System (Kernel) Space

On a 64-bit architecture, we can see that it utilizes a total of 14 TB of virtual memory (prior to Windows 8.1).  
The memory is split into User Process Space (Lower Addresses) and System Space (Higher Addresses).  
- 8 TB User Process Space
- 8 TB System (Kernel) Space

However, upon Windows 8.1 release, the virtual memory space expanded.
- 128 TB User Process Space
- 128 TB System (Kernel) Space

<img src="{{ site.url }}{{ site.baseurl }}/images/virtual-memory-layout-81.png" alt="">

If you noticed, the 64-bit architecture virtual memory layout has a section that's 'Unmapped'.  
This space is unmapped because of windows restrictions and limitations in CPUs, at this time.

General Architecture of Windows
---
Now that we've covered what processes, threads, objects and handles, and virtual memory.. let's go over the Windows general architecture.

<img src="{{ site.url }}{{ site.baseurl }}/images/general-arch.png" alt="">

In the above illustration, you'll see quite a few components that makeup the general architecture that Window's is built upon.

As you can see, based on the line separator, Windows is divided into two parts:
1. User Mode
2. Protected Mode

These two parts have several components associated to them.

Let's break these components down a little bit, just to get an idea of what each component is.

1. User Applications - These are processes that want to perform some kind of task on the operating system; such as CreateFile() via Notepad.exe.
2. Subsystem DLLs - In order to use the CreateFile() function described in the 'User Application' component, you need to use it's respected subsystem DLL that CreateFile() is implemented (in this case it'd be Kernel32.DLL).
3. NTDLL.DLL - This is the lowest layer in User Mode and is responsible for the transition from User Mode to Kernel Mode.
4. Executive - This is the upper layer in Kernel Mode, and is usually accessed by NTDLL.DLL upon transitioning from User Mode to Kernel Mode.
5. Kernel - The lower layer in Kernel Mode that provides important OS facilities; like Thread Scheduling and Interrupt and Exeception Dispatching.
6. Device Drivers - This a loadable kernel module that is responsible for carrying out operations for a specific device; such as file system and network drivers.
7. Hardware Abstraction Layer (HAL) - This is a key component in Windows for insulating between the hardware and software.
8. Graphics (Win32k) - This is used to handle all window related requests; such as resizing/minimzing a windows screen,etc. (UI related)
- For this, instead of being directed to the 'Executive' component, your request will be sent here
9. Environment subsystem - This component is a process that manages other subsystems (i.e. Win32 (CSRSS.EXE))
10. Services - This component manages processes running in User Mode (i.e. A Web Server listening for HTTP requests on TCP Port 80)
11. System Processes - This component manages important processes that are usually always running, and killing/crashing one of these processes can be fatal to the system (i.e. CSRSS.EXE, Services, SMSS.EXE)

Now that I've given an extremely high-level overview on each of the components, let me try and present all this information in a more visual way.

Function Call Flow (Part 1)
---
So how do we go from User Mode to Kernel Mode? What does it look like under the hood?

User mode can transition to Kernel mode when threads in user mode are performing operations (i.e. CreateFile()).

Let's dive a little deeeper into this process by following going through the components described in the previous section. Let's make use the following graphic to provide some visualization.

_Note: I will be demonstrating the flow of a function call on Windows 10 (x64) RedStone 2._

<img src="{{ site.url }}{{ site.baseurl }}/images/call-flow1.png" alt="">

When we request an operation; such as CreateFile(), the following process occurs:
Let's take a user application; such as Notepad.exe, and let's say it wants to create a new file.
- First, CreateFile() is called by the application process via a thread
	- Since CreateFileW() resides in the Kernel32.DLL subsystem, Kernel32 validates the CreateFile() parameters
- From Kernel32.DLL subsystem, since it doesn't have direct access to the CreateFile() object (Since it's in kernel mode), we call upon NTDLL.DLL
	- NTDLL.DLL is the lowest layer in User Mode, and is the intermediary between User Mode and Kernel Mode
- As stated before, NTDLL.DLL is responsible for the transition between User Mode and Kernel Mode, for the thread requesting service to CreateFile()
	- NTDLL.DLL has its own native api call for CreateFile() called 'NtCreateFile()'
	- This step is where we see the sysenter/syscall (x86/x64) call to jump from User Mode to Kernel Mode

To make a little more sense of this, let's take a look at these first few steps in WinDbg.

In order to debug an application in WinDbg, we'll open up Notepad.exe under the 'Open an Executable' option under the 'File' tab.

<img src="{{ site.url }}{{ site.baseurl }}/images/open-executable.png" alt="">

Once we attach WinDbg to Notepad, we'll see that we automatically get presented with a breakpoint.

From here, to demonstrate some of the other topics we've covered up to this point, we can look at the current threads running and its associated process for Notepad.exe, using the `~` command.

The process identifier for Notepad.exe (Using Task Manager):  

<img src="{{ site.url }}{{ site.baseurl }}/images/windbg-tilde-notepad-process.png" alt="">

The thread identifier for Notepad.exe (Using Process Explorer):

<img src="{{ site.url }}{{ site.baseurl }}/images/windbg-tilde-notepad-thread.png" alt="">

Let's proceed by setting a breakpoint on CreateFileW() in WinDbg. We can set this breakpoint with `bp Kernel32!CreateFileW` (or `bp KernelBase!CreateFileW`)
- KernelBase and Kernel32 go hand-in-hand
	- KernelBase recently came about in newer versions of Windows to improve efficiency and reduce surface area (Disk and memory requirements, etc.)
		- For more information on this, see ['New Low-Level Binaries'](https://docs.microsoft.com/en-us/windows/win32/win7appqual/new-low-level-binaries?redirectedfrom=MSDN) written by Microsoft.
- We use CreateFileW() over CreateFileA() or CreateFile() here because:
	- CreateFile() is not the actual function name (WinDbg will yell at you, if you try using it)
	- CreateFileA() is not used by Notepad (Notepad uses unicode functions by default)

After setting a breakpoint, we need to invoke an action within Notepad.exe that'll call the CreateFileW() function (In this case, we'll try saving a new text document)

Upon saving the new file, we hit our breakpoint and can dump the callstack to see what occured leading up to the CreateFileW() function.

<img src="{{ site.url }}{{ site.baseurl }}/images/notepad-callstack-createfilew.png" alt="">

From this point, we've seen what happens between the 'User Application - Notepad' and the 'Subsystem DLL - Kernel32.DLL'.  

Now let's see what happens between 'Subsystem DLL - Kernel32.DLL' and 'NTDLL.DLL'.  

Let's set a breakpoint on NtCreateFile with the `bp NTDLL!NtCreateFile`.

After successfully setting the breakpoint, and allowing execution to continue, we'll hit the breakpoint on NtCreateFile().  

If we dump the callstack with `k`, we can get a little more details on what happened leading up to NtCreateFile().

<img src="{{ site.url }}{{ site.baseurl }}/images/notepad-callstack-ntcreatefile.png" alt="">

Let's see a little more detail on what's going on in the NtCreateFile() function by using the unassemble command `u` and see what instructions are going to be executed.

<img src="{{ site.url }}{{ site.baseurl }}/images/notepad-ntcreatefile-unassembly.png" alt="">

Notice in the screen capture above the `mov eax, 55h` instruction. This instruction discloses the System Service Number (SSN) for the NtCreateFile() function.

A System Service Number (SSN) is a number in an array managed by the System Service Descriptor Table (SSDT).

When a program in User Space calls a function, in our case `Kernel32!CreateFileW`, eventually the execution of code is transferred to `NTDLL!NtCreateFile` in NTDLL.DLL.  
Then NTDLL.DLL will use `syscall` or `sysenter` to the kernel routine `Nt!NtCreateFile`.  

We'll cover a little more on System Service Numbers (SSNs) and the System Service Descriptor Table (SSDT) in the next section when we can see it in action.

If we keep tracing through the instructions, we'll see that we eventually hit the `syscall` instruction.

<img src="{{ site.url }}{{ site.baseurl }}/images/notepad-ntcreatefile-syscall.png" alt="">

Now, if we were to proceed with the execution of this function, we'll see that we don't see anything happening in Kernel Mode.  

Since we're debugging the application itself, we won't be able to analyze anything happening in Kernel Mode.

We'll analyze what's happening in Kernel Mode, coming up in the next section.

Function Call Flow (Part 2)
---
Now that we've seen what goes on in User Mode leading up to the transition from User Mode to Kernel Mode, let's go through what happens in Kernel Mode.

- After NTDLL.DLL transitions the thread's service request from User Mode to Kernel Mode, we reach the upper layer of Kernel Mode, known as 'Executive'
	- This layer is reached by calling a function within an NTDLL function
	- The 'Executive' consists of many components; such as the Memory and IO Managers, etc.
		- This layer also performs some security checks, but does nothing in terms of implementation of the thread service request
		- The handle is checked here, in order to call the appropriate driver, to continue with the service request
- The 'Executive' layer take the NtCreateFile() request received and passes it along to a lower level in Kernel Mode
	- It can be passed to one of two components: 'Device Drivers' or the 'Kernel'
		- Kernel: Provides important OS facilities (Thread Scheduling, Handling Interrupts, etc.)
		- Device Drivers: A loadable kernel module (something that can be created by developers)
	- In this example, the 'Executive' layer will pass the service request to the 'Device Driver' component
		- This is because CreateFile() function, since Windows utilizes the NTFS file system model, by default (NTFS.SYS)
- Once the service request is received by NTFS.SYS, the service request operation will commence
	- The device driver may need to go through the HAL
		- The HAL insolates the hardware from the software
			- An example is if a driver needs to register for an interrupt service, it doesn't need to get into the actual hardware (Interrupt Controller)
			- Instead the driver can go through the exposed functions provided by the HAL (but this isn't mandatory)

As we did in the previous section, let's try to make a little more sense of this flow using WinDbg.

In this situation, we won't simply be attaching to the Notepad.exe via the 'Open Executable' option in WinDbg.

For this, we'll be using LiveKD from the [Windows SysInternals Suite](https://live.sysinternals.com/).

If you're following along, let's amke sure your environment is setup correctly.

After downloading LiveKD, we need to place the module in the same directory WinDbg is kept.  

Upon placing LiveKD in the appropriate directory also containing WinDbg, we can proceed by opening an Administative Command Prompt in the directory containing both modules.

<img src="{{ site.url }}{{ site.baseurl }}/images/livekd-w.png" alt="">

Now that we're successfully debugging the live system, let's pick-up where we left off in the previous section.  

We've just identified the System Service Number (SSN) for NTDLL!NtCreateFile, and made it up to the `syscall` instruction.  

Let's look a little deeper into that System Service Number (SSN); which was fingerprinted as 55h in the `mov eax, 55h` instruction.

As mentioned before, the System Service Descriptor Table (SSDT) manages an array of addresses to kernel routines. These routines are index pointers to the NT system API calls; such as NT!NtCreateFile.  

Let's see if we can verify that the NTDLL!CreateFile System Service Number (SSN) is actually correct!

We can validate this by using WinDbg to analyze the System Service Descriptor Table (SSDT).

In WinDbg, we can see the Service Descriptor Table structure by using the `x nt!KiService*` command. This will provide us the information to the pointer of the SSDT itself - `NT!KiServiceTable`.

From here, we can dump the SSDT with the `dq NT!KiServiceTable` command. This command output will provide us with the relative offset to the kernel routines, at least on 64-bit architecture.

<img src="{{ site.url }}{{ site.baseurl }}/images/windbg-kiservice.png" alt="">

For us to get the absolute address of a kernel routine on a 64-bit system, we'll have to do the following illustrated in the visual below.

<img src="{{ site.url }}{{ site.baseurl }}/images/windbg-kiservicetable.png" alt="">

Using Nt!KiServiceTable and the SSN we identified earlier, we can calculate the relative offset to the kernel routine for `NT!NtCreateFile`.

Now that we have the relative offset, we can use that in the following formula to calculate the absolute address for `NT!NtCreateFile`:
- _'KiServiceTableAddress + (routineOffset >>> 4)'_  

As we can see, we successfully identified that the SSN for `NTDLL!NtCreateFile` correctly points to `NT!NtCreateFile`!

<img src="{{ site.url }}{{ site.baseurl }}/images/say-whattt.png" alt="">

We could use this same formula to calculate all the SSDT routines, if we wanted to. We can see this in action in the screenshot shown below.

<img src="{{ site.url }}{{ site.baseurl }}/images/enum-ssdt.png" alt="">

Below is another example on the flow of a function call for the ReadFile() function (alternate graphic format).
<img src="{{ site.url }}{{ site.baseurl }}/images/call-flow2.png" alt="">

References
---
If you're wanting to dive a little deeper into these topics that I've briefly covered in this post, I highly recommend checking out the following books:  
- [Windows Internals](https://www.amazon.com/Windows-Internals-Part-architecture-management-ebook/dp/B0711FDMRR/ref=sr_1_1?crid=2NO7YE7IA9N5H&keywords=windows+internals&qid=1585543596&sprefix=windows+intern%2Caps%2C151&sr=8-1) by Pavel Yosifovich, Alex Ionescu, Mark E. Russinovich, and David A. Solomon  
- [What Makes It Page?: The Windows 7 (x64) Virtual Memory Manager](https://www.amazon.com/What-Makes-Page-Windows-Virtual/dp/1479114294/ref=sr_1_1?keywords=what+makes+it+page&qid=1585543658&sr=8-1) by Enrico Martignetti

Wrapping Up
---
In this post, we managed to cover quite a few topics regarding Windows architecture and how memory is managed, at a high-level.  

While I'm still trying to fully grasp these topics at a lower-level, I hope I managed to provide you all with a decent amount of knowledge.  

I plan to keep building off this and provide you all with more content, in the near future.  

Please don't hesitate to reach out to me, if you have any questions, comments, and/or concerns about the topics I've covered here.  

Thanks!!

<img src="{{ site.url }}{{ site.baseurl }}/images/stay-classy.png" alt="">