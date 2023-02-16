# February 2023
## February 15th
### Embedded Xinu
Today I finished implementing the switch between user and kernel level processes in Embedded Xinu.  I met with Dr. Brylow and we came up with a more elegant solution that doesn't require the use of flags like I mentioned in yesterday's update.  Instead we add the following lines in `ctxsw.S`:
```armasm
	beq t0, ra, switch
	li t4, SSTATUS_S_MODE
    csrc sstatus, t4
	csrw sepc, t0

	sret
switch:
	ret
```
We use a heuristic to test if the process is running for the first time.  If the process hasn't run before, then it's `pc` (in `t0`) and `ra` would be different.  `pc` and `ra` are the same if the process is returning from a context switch.  If the process has already run before, that means its coming from a trap.  If it's coming from a trap, the code calls `ret` to avoid calling `sret` twice like mentioned in yesterday's update.  If the process hasn't run before, then the S-mode bit must be cleared so the process can run in U-mode when `sret` is executed.  With this implementation, the process control block no longer needs to keep track of it's privilege. 

### DNS Measurements
I worked on writing more of the background section and related works of the paper.  I hope to have both sections finished by next weekend.  Tomorrow I plan on testing the latest changes to the DNS scanner and setting up two more scanner instances in the U.S. 

## February 14th <3
### Embedded Xinu
I continued to work on the trap handler assignment today.  I talked to Dr. Brylow and he suggested a flag indicating if the process has run yet.  Basically, when we call resched, if the process has already run, we set the mode to s-mode (since it must have been called from a trap).  If the process hasn't run before, we keep it's mode at u-mode.  This allows processes to start in u-mode and remain there until a system call occurs.  This fixes the issues mentioned in yesterday's log.  The solution is mostly working, however I need to figure out if the main process test cases run on should be a user or system-level process.  If we make it a user level process, then we should make system calls for all functions we want the testcases to run.  Creating it as a user mode process seems "more correct" since it's not a core part of the OS.  Either way, I'll have to make code mondifications tomorrow to support whatever decision we make.

### TA-Bot
Jack and I finished the poster for SIGCSE today.  The final poster (as of today) can be found [here](/~agebhard/ta-bot/ta-bot-sigcse-2023-poster.pdf).  We plan to print it next week so we can ensure the graphics look good.

## February 13th
### Embedded Xinu
Today I worked on implementing the trap handler for the next assignment.  The trap handler is tricky because in the class version of Embedded Xinu we also have a seperate context switch file.  Both must be in sync to ensure we're in the correct mode (supervisor/user mode).  I have all the system calls working besides `user_yield()`.  `user_yield()` is tricky because the currently running process can switch in the middle of the trap.  If the trap handler switches modes from supervisor to user, and the context switch also switches modes, then the processor will end up in a state causing an Illegal Instruction exception (since we're trying to switch to user mode twice).  I haven't found a good fix for this yet, but I hope to tomorrow.

### TA-Bot
I worked with Jack today on the TA-Bot poster.  We need to submit it by tomorrow for the virtual SIGCSE participants.  We've got a rough draft done that we've showed Dr. Brylow.  We plan to make a few minor modifications before reviewing it one more time with him tomorrow.

## February 10th
### Compilers
In our Compilers meeting today, we discussed fixing some bugs in Dr. Brylow's reference implementation.  One of the main bugs that he found last semester was having issues loading strings in long programs.  In the current implementation of our compiler, the compiler puts strings in the data section of the file.  This works for most programs, however when we run BinaryString, the linker gives an error.  Dr. Brylow narrowed it down to attempting to load the string with `ldr`.  What we found was `ldr` allows for a range of +/-4096 bytes from the program counter according to the *Instruction Set Assembly Guide for Armv7 and earlier Arm® architectures* section C2.47.  This offset will work for most programs, but for BinarySearch, the linker failed to link the file because the string was outside of 4096 bytes.  To fix this, he added `movt`/`mov` instructions for all strings.  Dr. Brylow wants us to add a heuristic to the compiler to detect if the string is more than 4096 bytes away from the program counter.  If so, use the `movt`/`mov` opcodes.  Otherwise default to the `ldr` opcode.

### Embedded Xinu
Today I discussed platform support for Embedded Xinu.  We plan on still supporting the ARM architecture at least until it's clear if RISC-V will gain mainstream traction.  I want to merge the RISC-V and ARM code bases together.  To do this, I need to implement paging and memory protection in ARM.  I don't know how hard this will be since we've been working with RISC-V for the past year.

I also began working on the files for the next assignment, the trap handler.  We have to make some changes from how we implemented trap handlers in QEMU.  In QEMU, we set the `medeleg` register to handle S-mode environment calls in S-mode (a horizontal trap).  However, on the Nezha's, the `medeleg` is set by OpenSBI.  We have a few different options to handle this:
1. Modify OpenSBI to set the big on the `medeleg` register.
2. Modify U-Boot/OpenSBI to put the Nezha in M-Mode
3. Have processes run in U-Mode.

Option 3 is preferable since it doesn't require modification to U-Boot/OpenSBI.  I'm going to explore all three options on Monday.

## February 9th
### DNS Measurements
I put the final touches on my Advanced Computer Security presentation today.  I presented it and it went well!  [Here is the presentation](/~agebhard/dns-measurements/ImpersoNATION.pdf).  My hope for the project is to write a paper by the end of the semester on my findings.  I've found some relevant books and research papers on local government security by [Donald Norris from the University of Maryland](https://www.researchgate.net/scientific-contributions/Donald-F-Norris-2005216892).  I've ordered his book on local government security and plan to read his research paper's next week.

## February 8th
### Embedded Xinu
I spent most of the day finishing my lesson plan for the Operating Systems class today.  I believe the class went well!  I introduced the purpose of the context switch, Embedded Xinu's context switch, and went over the new files we gave the students to start the assignment.

### DNS Measurements
I continued to work on my presentation today for Advanced Computer Security tomorrow.  It's almost done and I plan to publish it tomorrow.

## February 7th
### Embedded Xinu
Today I worked on finalizing the assignment description and created a RISC-V cheatsheet for common opcodes in RISC-V.  That cheatsheet can be found [here](/~agebhard/embedded-xinu/RISC-V-Cheatsheet.pdf).  Everything should be ready for running the context switch assignment this week.  I also worked on writing a lesson plan for the Operating Systems class tomorrow since Dr. Brylow has a meeting.

### DNS Measurements
I've selected measuring SPF, DMARC, and CAA records for U.S. government domains as my Advanced Computer Security project.  For class Thursday, we are suppose to create a presentation describing our project.  I plan on sharing my current work as well as laying out a roadmap for the rest of the semester.

## February 6th
### Compilers
Today we discussed research papers we found in compilers.  I found a research paper ["Garbage collection for embedded systems" by Bacon *et al.*](https://dl.acm.org/doi/abs/10.1145/1017753.1017776).  The paper details two different garbage collection techniques: Mark-Coalesce-Compact and Paged Mark-Sweep-Defragment.  I didn't get to present my paper to the class today, so I'll go sometime later this week.

### Embedded Xinu
Most of my day was spent finishing up the tarball for the next assignment, Assignment #4 - Context Switch.  I fixed a bug in the previous assignment that prevented global variables from working.  We weren't clearing the .bss section when we started up the operating system, which caused errors when trying to access global variables that were initialzed to zeros.  I added the following lines in start.S to clear the .bss section:

```armasm
la a0, _bss
la t0, _end
sub a1, t0, a0
call bzero
```
I also began working on fixing TA-Bot to account for the difference in RISC-V and ARM.  The biggest difference for this assignment is ARM has 4 argument registers while RISC-V has 8.  I rewrote the big-args testcase to account for the difference.

## February 3rd
### Compilers
We worked on getting the last phase of our instruction selector in Embedded Xinu.  Jake added his instruction selector so we can start working on implementing an instructor selector for RISC-V.  We plan on fixing up his instruction selector before we move on into adding RISC-V support.

## February 2nd
*No significant research activity happened today.*

## February 1st
### Compilers
Today we met to discuss garbage collection and how we would go about implementing garbage collection in our MiniJava compiler.  We are looking at implementing a stop-the-world garbage collector.  We are currently researching different garbage collection techniques to possibly implement in Embedded Xinu.  We plan to have research articles on garbage collection that we can discuss next Monday.