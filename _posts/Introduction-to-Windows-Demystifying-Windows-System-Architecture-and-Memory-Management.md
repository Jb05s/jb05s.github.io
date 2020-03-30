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

Kernel Mode vs User Mode
---
<img src="{{ site.url }}{{ site.baseurl }}/images/privilege-rings.png" alt="">

In the illustration shown above, there's a total of seven layers (or rings) of protection.

Whenever code executes, it always will have a mode associated with it. This is called the "Thread Access Mode" or better known as the "Current Privilege Level (CPL)"

Windows only uses two of those rings:
1. Ring 0 (Kernel Mode); which is the most privileged
2. Ring 3 (User Mode); which is the least privileged

User Mode (CPL3)
---
This mode doesn't allow access to operating system code or data, and is denied access to system hardware.
If a crash occurs in this mode, the system is not effected, only in the application where the error occurred.

Kernel Mode (Privileged) (CPL 0)
---
In this mode, it has complete access to the kernel and device drivers.
Additionally, this mode is allowed to access all system resources.
Any unhandled exception in kernel mode can result in a system crash, infamously known as the Blue Screen of Death (BSoD).

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

Based on the CPL you're operating in, you'll have the ability to read and write data in the segments of that CPL and of that of the lesser.

I'll shine some more light on this later in the post, when I talk about the flow of a function call.

Processes
---
Processes are management containers for threads to execute code. 
There's multiple occasions where I've heard "There's a process running..". This is an inaccurate statement. A process is never "running". 
A process is simply a container (or manager) resourcing threads. Threads are actually "running" or executing code, not processes.
A process consists of the following:
- A private virtual address space
- An executable program containing data that can be executed
- A table of handles referencing kernel objects
- An access token; Used for security checks when accessing shared resources
- Manage one or more threads; the entity that performs code execution

(Process Explorer visuals)

Threads
---
Threads, as mentioned earlier, are entities scheduled by the kernel to execute code.
- Multiple threads can run concurrently on multiple CPUs
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


(Process Explorer visuals)

Objects and Handles
---
Objects are runtime instances of static structures
	- i.e: Process, mutex, event, desktop, file
Reside in system memory space
Kernel code can obtain direct pointer to an object.
In order for user mode code to get access to an object is by using a handle.
A handle is an index in a table that points to a specifc object in kernel space
	- Shields user code from directly accessing an object
	- Handle values are always in multiples of 4
Objects are reference counted; so close the handle, after completing a task with the object
	- The Object Manager is responsible for managing the reference counter

(Process Explorer Visuals - See Windows Internals 'Demo: Objects and Handles')

Virtual Memory
---
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

Virtual Memory Layout
---
4 GB - 32bit (2GB User Process Space (0x00000000 - 0x7FFFFFFF) / 2GB System Space (0x80000000 - 0xFFFFFFFF))  
256 TB - 64bit (1288TB User Process Space (0x00000000'00000000 - 0x00007FFF'FFFFFFFF) / 128TB System (Kernel) Space (0xFFFF0800'00000000 - 0xFFFFFFFF'FFFFFFFF))
- The remainder of memory space for x64 is unmapped, due to Windows restrictions and/or due to limitations in CPUs

In task manager or process explorer, under the 'Details' tab, you can see the 'Memory (Private Working Set)' column.
- 'Private' meaning the memory is not shared
	- Allocating memory via 'malloc' is considered private memory, this is what is shown
	- Loading a DLL is sharable memory, and isn't shown in this column but in the 'Memory (Shared Working Set)' column
		- To get a better idea of how much memory a process consumes, view the 'Commit Size' column
- 'Working Set' is a term used to describe physical memory
	- This column shows how much memory is being used by the process
- The values under this column are always divisible by 4 (The size of a memory page)

(Windbg visuals / Process Explorer (or Task Manager) Visuals / VMMap visuals)

General Architecture of Windows
---
Now that we've covered what processes, threads, objects and handles, and virtual memeory. Let's first go over the Windows general architecture.

<img src="{{ site.url }}{{ site.baseurl }}/images/general-arch.png" alt="">

In the above illustration, you'll see quite a few components that makeup the general architecture that Window's is built upon.

As you can see, based on the line separator, Windows is divided into two parts:
1. User Mode
2. Protected Mode

User mode consists of quite a few components; such as several Subsystems, Services, DLLs, etc. Let's break these components down a little bit, just to get an idea of what each component is.

1. User Applications - These are processes that want to perform some kind of task on the operating system; such as CreateFile() via Notepad.exe.
2. Subsystem DLLs - In order to use the CreateFile() function described in the 'User Application' component, you need to use it's respected subsystem DLL that CreateFile() is implemented (in this case it'd be Kernel32.DLL).
3. NTDLL.DLL - This is the lowest layer in User Mode and is responsible for the transition from User Mode to Kernel Mode.
4. Executive - This is the upper layer in Kernel Mode, and is usually accessed by NTDLL.DLL upon transitioning from User Mode to Kernel Mode.
5. Kernel - The lower layer in Kernel Mode that provides important OS facilities; like Thread Scheduling and Interrupt and Exeception Dispatching.
6. Device Drivers - This a loadable kernel module that is responsible for carrying out operations for a specific device.
7. Hardware Abstraction Layer (HAL) - This is a key component in Windows for insulating between the hardware and software.
8. Graphics (Win32k) - This is used to handle all window related requests; such as resizing/minimzing a windows screen,etc. (UI related)
- For this, instead of being directed to the 'Executive' component, your request will be sent here
9. Environment subsystem - This component is a process that manages other subsystems (i.e. Win32 (CSRSS.EXE))
10. Services - This component manages processes running in User Mode (i.e. A Web Server listening for HTTP requests on TCP Port 80)
11. System Processes - This component manages important processes that are usually always running, and killing/crashing one of these processes can be fatal to the system (i.e. CSRSS.EXE, Services, SMSS.EXE)

Now that I've given an extremely high-level overview on each of the components, let me try and present all this information in a more visual way.

Function Call Flow
---
So how do we go from User Mode to Kernel Mode? What does it look like under the hood?

User mode can transition to Kernel mode when threads in user mode are performing operations (i.e. CreateFile()).

Let's dive a little deeeper into this process by following going through the components described in the previous section. Let's make use the following graphic to provide some visualization.

<img src="{{ site.url }}{{ site.baseurl }}/images/call-flow1.png" alt="">

When we request an operation; such as CreateFile(), the following process occurs:
Let's take a user application; such as Notepad.exe, and let's say it wants to create a new file.
- First, CreateFile() is called by the application process via a thread
	- Since CreateFileW() resides in the Kernel32.DLL subsystem, Kernel32 validates the CreateFile() parameters
- From Kernel32.DLL subsystem, since it doesn't have direct access to the CreateFile() object (Since it's in kernel mode), we call upon NTDLL.DLL
	- NTDLL.DLL is the lowest layer in User Mode, and is the intermediary between User Mode and Kernel Mode
- As stated before, NTDLL.DLL is responsible for the transition between User Mode and Kernel Mode, for the thread requesting service to CreateFile()
	- NTDLL.DLL has its own native api call for CreateFile() called 'NtCreateFile()'
- After NTDLL.DLL transitions the thread's service request from User Mode to Kernel Mode, we reach the upper layer of Kernel Mode, known as 'Executive'
	- This layer is reached by calling a function within an NTDLL function
	- The 'Executive' consists of many components; such as the Memory and IO Managers, etc.
		- This layer also performs some security checks, but does nothing in terms of implementation of the thread service request
		- The handle is checked here, in order to call the appropriate driver, to continue with the service request
	- The 'Executive' layer take the NtCreateFile() request received and passes it along to a lower level in Kernel Mode
		- It can be passed to one of two components: 'Device Drivers' or the 'Kernel'
			- Kernel: Provides important OS facilities (Thread Scheduling, Handling Interrupts, etc.)
			- Device Drivers: A loadable kernel model (something that can be created by developers)
		- In this example, the 'Executive' layer will pass the service request to the 'Device Driver' component
			- This is because CreateFile() function, since Windows utilizes the NTFS file system model, by default (NTFS.SYS)
	- Once the service request is received by NTFS.SYS, the service request operation will commence
		- The device driver may need to go through the HAL
			- The HAL insolates the hardware from the software
				- An example is if a driver needs to register for an interrupt service, it doesn't need to get into the actual hardware (Interrupt Controller)
					- Instead the driver can go through the exposed functions provided by the HAL (but this isn't mandatory)

Below is another example for the ReadFile() function call.
<img src="{{ site.url }}{{ site.baseurl }}/images/call-flow2.png" alt="">