---
title: Quiz Day 1
teaching: 30
exercises: 0
questions:
- “Do you know your stuff?"
objectives:
- "Validate your understanding by working through some examples."
keypoints:
- “Practice makes perfect."
---

**Quiz Time**

> #### Question 01: What is the key difference between Python scripts and modules?
> * scripts are meant to be imported, while modules are made to be directly executed.  
> * modules are meant to be imported, while scripts are made to be directly executed.  
>
> > ## Solution
> > **Solution:**  A plain text file containing Python code that is intended to be directly executed by the user is usually called script, which is an informal term that means top-level program file. On the other hand, a plain text file, which contains Python code that is designed to be imported and used from another Python file, is called module.
> {: .solution}
> {: .challenge}

> #### Question 02: What is the key difference between Python scripts and modules?What are some of the ways we can use to run Python code interactively? Select all that apply:  
> A: macOS: Entering python3 in the Terminal.  
> B: Linux: Entering python3 in the Terminal window (depending on the Linux distribution).  
> C: Windows: Entering python is the command prompt.  
> D: Windows: Entering start python in the Windows Run dialog.  
> E: macOS: Entering python run in the Terminal.  
> F: Linux: Entering python execute in the Terminal window (depending on the Linux distribution).  
>
> > ## Solution
> > **Solution:** The solution to this question includes items A, and B.
> {: .solution}
> {: .challenge}

> #### Question 03: When you try to run Python scripts, a multi-step process begins. In this process the interpreter performs three steps:  
> 1. Ship off the code for execution.  
> 2. Process the statements of your script in a sequential fashion.  
> 3. Compile the source code to an intermediate format known as bytecode.  
>
> Identify the correct order of these steps:  
> * 1 → 3 → 2  
> * 2 → 3 → 1  
> * 3 → 2 → 1  
> * 1 → 2 → 3  
>
> > ## Solution
> > **Solution: 2 → 3 → 1**  
> > When you try to run Python scripts, a multi-step process begins. In this process the interpreter will:  
> > Step 1: Process the statements of your script in a sequential fashion.  
> > Step 2: Compile the source code to an intermediate format known as bytecode. This bytecode is a translation of the code into a lower-level language that’s platform-independent.  
> > Step 3: Ship off the code for execution. At this point, something known as a Python Virtual Machine (PVM) comes into action. The PVM is the runtime engine of Python.  
> > It is a cycle that iterates over the instructions of your bytecode to run them one by one.  
> > The whole process to run Python scripts is known as the Python Execution Model.  
> {: .solution}
> {: .challenge}

> #### Question 04: The following executable Python code, stored as a file, doesn’t execute from a file manager on a Linux machine.  
> print("Welcome to Real Python!")  
> What should we add to the first line of the file to ensure it runs? Select all that apply.  
>
> A) `#!/usr/bin/python`  
> B) `#!/usr/bin/env python3`  
> C) `#!/usr/bin/env python`  
> D) `#!/usr/bin python`  
>
> > ## Solution: 
> **Solution:** A, B and C  
> To make sure your Python file runs, remember to specify the Python path to the interpreter. It can be done by adding the path to the first line of your file. There are two ways to do so:  
>  *  `#!/usr/bin/python:` Writing the absolute path.  
>  * `#!/usr/bin/env python` or `#!/usr/bin/env python3`: Using the operating system env command, which locates and executes Python by searching the PATH environment variable.  
> {: .solution}
> {: .challenge}

> #### Question 05:  What is the output of L[1:] if L = [1,2,3]?
> * A: 2, 3
> * B: 2
> * C: 3
> * D: None of the above.
>
> > ## Solution
> **Solution:** A  
> {: .solution}
> {: .challenge}

> #### Question 06: What mistake is present in this sequence of commands?  
> `setup larsoft v09_22_02 -q e19:prof`  
> `mrb newDev -q e19:prof`  
> `source localProducts*/setup`  
> `cd $MRB_SOURCE`  
> `mrb g dunetpc`  
>  
> A: `dunetpc` doesn’t depend on larsoft  
> B: the head of develop of `dunetpc` is newer than `v09_22_02` so you’ll get a version mismatch between larsoft and dunetpc  
> C: sourcing with a wildcard is not allowed  
> D: none of the above.  
>
> > ## Solution
> **Solution:** 
> > 
> {: .solution}
> {: .challenge}

> #### Question 07: How do you fix the mistake in Question 06?  
> A: `git checkout v09_22_02 in the dunetpc source tree`  
> B: `use mrb g -t v09_22_02 dunetpc instead of the tagless version above`  
> C: log out and log back in again but don’t setup larsoft -- ups will do it for you when you set up the local dunetpc  
>
> > ## Solution
> **Solution:** A, B, or C will do it 
> > 
> {: .solution}
> {: .challenge}

> #### Question 08: What is the output of this sequence of C++ statements?  
> `size_t a=5;`  
> `size_t b=6;`  
> `std::cout << a-b << std::endl;`  
>
> A: 1  
> B: -1  
> C: 18446744073709551615  
> D: 0  
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}


> #### Question 09: What happens when the compiler emits a warning when building software with `mrb`?  
> A: They are printed to the screen and you can choose to ignore them.  
> B: They are considered to be errors and will stop your build.  
> C: They are not printed to the screen and the build will proceed.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 10: A build with `mrb` fails to finish due to an error in a source file.  Which of these is true?
> A: Your `localProducts` directory now has an incomplete set of shared libraries and you cannot proceed until you fix the problem.
> B: Your `localProducts` directory contains the results of the most recent successful build, or it may be empty if you had no prior successful builds.
> C: Your `localProducts` directory is empty, regardless of what was there before. 
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 11: When should you initialize variables that count objects (like tracks or clusters?)  
> A: You don’t have to initialize counters because the compiler will do it for you.  
> B: You should initialize counters only when they are defined.  
> C: When they are defined, and as needed (e.g. at job start time if you want to count all objects in all events, and at event processing start time if you just want to count the objects in each event).  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 12: What is a segmentation fault? 
> A: It is the result of an invalid memory access.  
> B: It is an indication that your program has run out of disk space.  
> C: It is an indication that your program has run out of memory.  
> D: It is an error message saying you should divide your program into different segments.   
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 13: What is the output of these C++ statements?  
> `float a=4.99;`  
> `int b=a;`  
> `std::cout << b << std::endl;`  
>
> A: 5  
> B: 4.99  
> C: 4  
> D: 4.0  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}


> #### Question 14: What is the output of these C++ statements?
> `float a=-4.99;`  
> `int b=a;`  
> `std::cout << b << std::endl;`  
>
> A: -5  
> B: -4.99  
> C: -4  
> D: -4.0  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}


> #### Question 15: You want to store a list of hit ID numbers so you can process them later in your program.  Which of these is the best choice?  
> A: Define an array with a fixed-size dimension that’s at least as big as the biggest number of hits you expect to see.  
> N: Allocate memory on the heap with new.  
> C: Allocate memory on the stack with new.  
> D: Use `std::vector<size_t>` in order to let the list grow.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 16: 
> A: 
> B: 
> C: 
> D: 
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 17: You are reading in a data file and notice that the kinetic energy of a particle is recorded as <i>-999 GeV</i>.  What’s that?  
> A: That’s a “guard value” to indicate invalid or missing data  
> B: Heisenberg’s Uncertainty Principle means that’s okay.  Go ahead and include it in your total energy sum.  
> C: You may have a memory overwrite -- memory interpreted as a floating-piont number but which had an int or a string written to it can produce crazy answers.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 18: You are reading in a data file and notice that the kinetic energy of a particle is recorded as <i>5.23E212 GeV</i>.  What is a likely explanation?  
> A: That’s a “guard value” which indicates invalid or missing data.  
> B: Heisenberg’s Uncertainty Principle means that’s okay.  Go ahead and include it in your total energy sum.  
> C: You may have a memory overwrite -- memory interpreted as a floating-piont number but which had an int or a string written to it can produce crazy answers.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 19: You have an intermittent bug -- sometimes when you run your program it crashes, and sometimes when you run it with the same inputs, it succeeds.  What is a likely cause?  
> A: Your computer is broken and you need a new one.  
> B: Your program may depend on the value stored in uninitialized memory.  Time to run valgrind!  
> C: Your program is sensitive to the value of the output of a random number generator.  
> D: B and C are likely  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 20: Which of the following are examples of “defensive programming”?  
> A: Check all input data for validity before attempting to process it.  
> B: Check return status codes from methods that may fail.  
> C: Cut and paste example code from web sites without reading it.  
> D: Check for null pointers before dereferencing them.  
> E: A, B, and D  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 21: ROOT files that are partially written by a program that crashed can always be recovered.  
> A: True  
> B: False  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 22: You have a big, constant rootfile you need to input to all your jobs, like a giant magnetic field map or a detector response lookup library.  It is 100 MB big.  How do you store it and distribute it?  
> A: Check it in to github  
> B: Check it in to git on Redmine  
> C: Put it in your home directory  
> D: Put it in `StashCache`  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 23: What happens when an exception is thrown in C++?  
> A: The program terminates at the line that threw the exception.  
> B: The debugger is invoked.  
> C: A stack trace is printed out and execution continues.  
> D: The stack is unwound until a caller catches the exception.  If uncaught, the program terminates.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 24: What happens if you try to use `TFile::Get()` on an object that doesn’t exist in a `Rootfile` (say you misspelled its name)?  
> A: ROOT throws an exception and stops.  
> B: ROOT prints out a message saying “Object not found”  
> C: The result of the Get() call is a null pointer and execution continues without a message.  
> D: ROOT gives you a pointer to the last object you successfully retrieved from the file.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 25: What is the difference between a `std::map` and a `std::unordered_map`?  
> A: `td::map` sorts the elements by key when they are inserted and `std::unordered_map` does not.  
> B: `std::unordered_map` uses a hash table to look up entries while std::map uses a binary search.  
> C: `std::map` finds elements in `O(logN)` time and `std::unordered_map` finds elements in `O(1)` time.  
> D: `std::unordered_map` takes more memory due to the hash table.  
> E: all of the above  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 26: Which of the following data types is a good choice for numbering the TPC channels number in a single-phase far detector module?  
> A: `size_t`  
> B: `int`  
> C: `uint32_t`  
> D: `char`  
> E: `short`  
> F: A, B and C will work.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 27: Approximately how many decimal digits of precision are available in a floating-point number?  
> A: 7  
> B: 12  
> C: 3  
> D: 20  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 28: Your module needs to use the average drift velocity of electrons in liquid argon.  What’s the best way to get this number?  
> A: Hard-code it in your source file.  
> B: Pass it in as a fcl parameter.  
> C: See if it is defined elsewhere (e.g. in a service that looks at a fcl parameter or a database) and use that one.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 29: What kinds of storage classes are available in `dCache`?
> A: Scratch, Persistent, Tape-Backed and Resilient  
> B: Scratch, Persistent and Solid-State  
> C: Tape-Backed, Persistent and Auto-Cataloged  
> D: Read-only, Read/Write and Write-only  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 30: Why do we use both `gcc` and `clang` to compile C++ programs?  
> A: To take up more space on the disks.  
> B: To get different compiler warnings and errors about possibly mistaken code.  
> C: They’re both free, so we save more money by using both.  
> D: To see which one compiles the code the fastest.  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 31: You take the logarithm of -2.  What is the result?   
> A: `0`  
> B: `NaN`  
> C: `Inf`  
> D: `32767`  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 32: The qualifiers “prof” and “debug” mean:  
> A: Professor of entomology  
> B: Whether profiling is enabled at compile time or not  
> C: Whether debug symbols are included in the shared libraries or not  
> D: Different compiler optimization settings  
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 33: What’s the best way to get ROOT documentation?  
> A: Google it (or use your favorite search engine)  
> B: Read the ROOT manual.  
> C: `grep -r -i keyword $ROOTSYS/tutorials`  
> D: All of these work, but some are slower than others (but you might find other useful things).
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}

> #### Question 34: 
> A:  
> B:  
> C:  
> D:   
>
> > ## Solution
> **Solution:**
> > 
> {: .solution}
> {: .challenge}


{%include links.md%}
