# February 2023
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