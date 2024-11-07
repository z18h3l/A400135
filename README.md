java c
A4: Virtual Memory
Due Nov 12 by 11:59p.m.
Points 90
Available after Oct 23 at 12a.m.
Introduction
In this assignment, you will investigate memory access patterns, simulate the operation of page tables and implement several page replacement algorithms. This will give you some practice working with the algorithms we have been talking about in class.
This assignment is based on a virtual memory simulator that uses the simvaddr-*.ref memory reference traces located at /u/csc369h/fall/pub/a4/traces. The first task is to implement virtual-to-physical address translation and demand paging using a page table design of your choice. Then you will implement two different page replacement algorithms: simplified 2Q and Clock.
Before you start work, you should complete the set of readings about memory, if you haven't done so already:
Paging: Introduction (http://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf)
Background
Valgrind has an option that will allow you to print out the memory reference trace of a running program (using the lackey tool), which we are using to generate traces for the virtual memory simulator in Assignment 4.
There are five different C programs that we've created address traces, and they are used in the virtual memory simulator for this assignment. The programs are:
simpleloop - loops over an array allocated in the heap (You can also modify the code to run the same loop using stack memory)
repeatloop - loops over a heap-allocated array multiple times
matmul - naive matrix multiply with the ability to change the element size to change the memory access behaviour
blocked - a more memory-aware matrix multiply that should exhibit a better hit rate under some page replacement algorithms
reuse_scan - repeatedly accesses a smaller array while sequentially accessing every page in a very large array (should showcase the benefits of a scan-resistant algorithm like simplified 2Q)
We have provided the half-processed traces from Valgrind in the addr-*.ref memory reference traces files in case you are interested. These files are not used by your simulator.
Part 1: Virtual to physical translation
Fall 2023 Intro Video (https://utoronto-my.sharepoint.com/:v:/g/personal/angela_demkebrown_utoronto_ca/EbKyHJpzWPxPhps-NQ2_Y4sBnQPBOPBgaYVQEC8edO06Bg?e=GTXGbm)
Setup
Log into MarkUs to create or update your repo and get the starter code. Remember that you cannot manually create a new a4 directory in your repo or MarkUs won't see it.
The traces from our sample programs at /u/csc369h/fall/pub/a4/traces will be interesting to run once you have some confidence that your program is working, but you will definitely want to create small traces by hand for testing.
The format of the traces is reftype vaddr value as shown in the sample below. Note that the page offset part of the addresses are all between 0 and 15 (0xf) to fit in the reduced simulated physical page frames. For a write reference type (S or M), the value will be written to the virtual address. For a read reference type (L or I) the value is the expected value that should be read from the virtual address. It should always be the same as the value in the most recent preceding write reference to the same virtual address. We use this to check that the address translations and pagein/pageout operations are working correctly. A sample trace snippet is shown below:
S 309001 182
S 1fff000000 55
I 108005 0
S 308008 122
L 1fff000000 55
L 308008 122
I 4cc5000 0
L 5018008 0
Note that in our traces, the Instruction reference type is likely to always have a value of 0 because these addresses are not written to after the program starts executing. You will also see Load references with a value of 0 when the trimmed trace includes a Load from an address that has not yet been written to.
malloc369
We have provided a custom malloc library named malloc369 which can help you with detecting memory leaks, and also ensure that you do not use too much memory for your page table implementation. Make sure you only allocate memory using malloc369() and free dynamically allocated memory using free369(). Do not use the C library malloc function directly, as it bypasses our dynamic memory tracker.
Task 1 - Address Translation and Paging
Implement virtual-to-physical address translation and demand paging using a pagetable design of your choice.
The main driver for the memory simulator, sim.c, reads memory reference traces in the format shown above, from trimmed, reduced valgrind memory traces. For each line in the trace, the program asks for the simulated physical page frame. that corresponds to the given virtual address by calling find_physpage, and then reads from the simulated physical memory at the location given by the physical frame. number and the page offset. If the access type is a write ('M' for modify or 'S' for store), it will also write the value from the trace to the location. You should read sim.c so that you understand how it works but you should not modify it.
The simulator is executed as ./sim -f  -m  -s  -a  where memory size and swapfile size are the number of frames of simulated physical memory and the number of pages that can be stored in the swapfile, respectively. The swapfile size should be as large as the number of unique virtual pages in the trace.
There are four main data structures that are used:
1. unsigned char *physmem: This is the space for our simulated physical memory. We define a simulated page frame. size of SIMPAGESIZE and allocate SIMPAGESIZE * "memory size" bytes for physmem.
2. struct frame. *coremap: The coremap array represents the state of (simulated) physical memory. Each element of the array represents a physical page frame. It records if the physical frame. is in use and, if so, a pointer to the page table entry for the virtual page that is using it.
3. struct pt_entry - a page table entry. The format of a page table entry is up to you, but at a minimum it must record 代 写A4: Virtual MemoryR
代做程序编程语言the frame. number if the virtual page is in (simulated) physical memory and an offset into the swap file if the page has been written out to swap. It must also contain flags to represent whether the entry is Valid, Dirty, and Referenced.
4. swap.c: The swapfile functions are all implemented in this file, along with bitmap functions to track free and used space in the swap file, and to move virtual pages between the swapfile and (simulated) physical memory. The swap_pagein and swap_pageout functions take a frame. number and a swap offset as arguments. The simulator code creates a temporary file in the current directory where it is executed to use as the swapfile, and removes this file as part of the cleanup when it completes. It does not, however, remove the temporary file if the simulator crashes or exits early due to a detected error. You must manually remove the swapfile.XXXXXX files in this case.
To complete this task, you will have to write code in pagetable.c and pagetable.h. Read the code and comments in this file -- it should be clear where implementation work is needed and what it needs to do. Basic round-robin and random replacement algorithms are already implemented for you, so you can test your translation and paging functionality independently of implementing the replacement algorithms.
Efficiency: In a real operating system implementation, the memory space taken up for your page tables reduces the memory space available to store the pages of processes' virtual address spaces. Hence, keeping page tables small is desirable. Reducing the time complexity of page table lookups is also important. Your solution will be evaluated on correctness as well as space and time efficiency.
Task 2 - Replacement Algorithms
Using the starter code, implement CLOCK (with one ref-bit) and simplified 2Q replacement algorithms.
Note: to test your page replacement algorithms, we will replace your pagetable.c with a solution version, so your page replacement algorithm must be contained to the provided functions.
Once you are done implementing the algorithms you can use the provided simvaddr-*.ref traces and the autotester to check the results. For each algorithm, the tester will run the programs on memory sizes 50 and 100 and check the output against the expected results.
Efficiency: Page replacement algorithms must be fast, since page replacement operations can be critical to performance. Consequently, you must implement these policies with efficiency in mind. For example, we will give you the expected complexities for some of the policies:
RR: init, evict, ref: O(1) in time and space
CLOCK: init, ref: O(1) in time and space; evict: O(M) in time, O(1) in space, where M = size of memory
simplified 2Q: init, evict, ref: O(1) in time and space
Important notes
When we run the autotests on your code, your page replacement algorithms will be compiled with a different pagetable.c file (the one from the solution). All the code of the page replacement algorithms must be in their separate .c files, not in pagetable.c.
When a page is being evicted, there should be only 2 possibilities: (i) the page is dirty and needs to be written to the swap; and (ii) the page is clean and already has a copy in the swap. A newly initialized page (zero-filled) should be marked dirty on the very first access.
CLOCK must use the "Referenced" flag stored in the page table entry. All the algorithms must utilize their ref() functions (if necessary) instead of adding any algorithm-specific code to pagetable.c. There are functions in pagetable.c to get the values of the Valid, Dirty, and Referenced flags given a page table entry, which you should implement. Use these functions in your replacement algorithm implementations if you need to check any of these flags -- do not assume a particular format for page table entries, or your replacement algorithms are unlikely to work with our solution version of pagetable.c.
The simulator and the page replacement algorithms must not produce any additional or different (from starter code) output (except for errors that should be printed to stderr), otherwise the tests will fail.
The simulator writes a value into the simulated physical memory pages for Store or Modify references, and checks that simulated physical memory contains the last written value on Load or Instruction references. If there is a mismatch, the simulator prints an error message. These errors indicate that there is something wrong with the address translation implementation.
For debugging, you will find it useful to implement the print_pagetable() function in pagetable.c. The function should print (at least) one line for each in-use page in the pagetable (valid in memory, or currently evicted to swap). Other than that, what information it displays is up to you -- we will only be testing that it produces at least the expected number of outputs lines.
You can use the debug variable in sim.c to control extra output during development (you can use the -d  option to sim.c to selectively print different levels of detail in your debug output, e.g. './sim -d 1 -f ...'). During grading, we will run sim with debug = 0.
Submission
Once you have tested your code and verified that it works according to specification, and committed it locally (check that by running git status), you can git push it back to MarkUs. We will collect and grade the last version pushed to MarkUs after the assignment deadline. Note that whatever you get from the autotester will be your final mark, therefore, you are recommended to run it before your final submission.
You must only modify the following files:
pagetable.c
pagetable.h
clock.c
s2q.c
In addition, you may modify list.h, if you do not want to use the provided linked list implementation. Read the comments in that file before making any changes.
You may push other code to the repository, but the tester will ignore them. Lastly, please do not push tester output files into the repository.





         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
