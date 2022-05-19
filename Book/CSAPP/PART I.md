## 1.Information Is Bits + Context

​     The representation of hello.c illustrates a fundamental idea: All information in a system—including disk fifiles, programs stored in memory, user data stored in memory, and data transferred across a network—is represented as a bunch of bits. The only thing that distinguishes different data objects is the **context** in which we view them. For example, in different contexts, the same sequence of bytes might represent an integer, flfloating-point number, character string, or machine instruction.



So why C success?

- **C was closely tied with the Unix operating system**. C was developed from the beginning as the system programming language for Unix. Most of the Unix kernel (the core part of the operating system), and all of its supporting tools and libraries, were written in C. As Unix became popular inuniversities in the late 1970s and early 1980s, many people were exposed to C and found that they liked it. Since Unix was written almost entirely in C, it could be easily ported to new machines, which created an even wider audience for both C and Unix.
- **C is a small, simple language**.The design was controlled by a single person, rather than a committee, and the result was a clean, consistent design with little baggage. The K&R book describes the complete language and standard library, with numerous examples and exercises, in only 261 pages. The simplicity of C made it relatively easy to learn and to port to different computers.
- **C was designed for a practical purpose**. C was designed to implement the Unix operating system. Later, other people found that they could write the programs they wanted, without the language getting in the way.

​	C is the language of choice for system-level programming, and there is a huge installed base of application-level programs as well. However, it is not perfect for all programmers and all situations. **C pointers are a common source of confusion and programming errors. C also lacks explicit support for useful abstractions such as classes, objects, and exceptions.** Newer languages such as C++ and Java address these issues for application-level programs.



## 2.Programs Are Translated by Other Programs into Different Forms



<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503120101950.png" alt="image-20220503120101950" style="zoom: 67%;" />



​	The programs that perform the four phases (*preprocessor*, *compiler*, *assembler*, and *linker*) are known collectively as the *compilation system*. ）

- **Preprocessing phase**
- **Compilation phase**
- **Assembly phase**
- **Linking phase**



## 3.It Pays to Understand How Compilation Systems Work

However, there are some important reasons why programmers need to understand how compilation systems work:

- **Optimizing program performance**
- **Understanding link-time error**
- **Avoiding security holes**



## 5.Caches Matter

​	Because of physical laws, larger storage devices are slower than smaller storage devices. And faster devices are more expensive to build than their slower counterparts. Similarly, a typical register file stores only a few hundred bytes of information, as opposed to billions of bytes in the main memory. However, the processor can read data from the register file almost 100 times faster than from memory. Even more troublesome, as semiconductor technology progresses over the years, this processor–memory gap continues to increase.  It is easier and cheaper to make processors run faster than it is to make main memory run faster.

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503152424135.png" alt="image-20220503152424135" style="zoom:67%;" />

​	To deal with the processor–memory gap, system designers include smaller, faster storage devices called cache memories (or simply caches) that serve as temporary staging areas for information that the processor is likely to need in the near future. **L1 and L2 cache**

​	The idea behind caching is that a system can get the effect of both a very large memory and a very fast one by exploiting **locality**, the tendency for programs to access data and code in localized regions. By setting up caches to hold data that are likely to be accessed often, we can perform most memory operations using the fast caches.

## 6.Storage Devices Form a Hierarchy



<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503152447703.png" alt="image-20220503152447703" style="zoom:50%;" />



## 7.The Operating System Manages the Hardware

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503153429651.png" alt="image-20220503153429651" style="zoom: 67%;" />

​	All attempts by an application program to manipulate the hardware must go through the operating system.

The operating system has two primary purposes:

1. **to protect the hardware from misuse by runaway applications**.
2. **to provide applications with simple and uniform mechanisms for manipulating complicated and often wildly different low-level hardware devices**. 

​	The operating system achieves both goals via the fundamental abstractions shown in Figure 1.11: processes, virtual memory, and files. As this figure suggests, files are abstractions for I/O devices, virtual memory is an abstraction for both the main memory and disk I/O devices, and processes are abstractions for the processor, main memory, and I/O devices. We will discuss each in turn.

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503153922039.png" alt="image-20220503153922039" style="zoom:67%;" />

### 7.1 Process

​	A process is the operating system’s abstraction for a running program.

​	Traditional systems could only execute one program at a time, while newer multicore processors can execute several programs simultaneously. In either case, a single CPU can appear to execute multiple processes concurrently by having the processor switch among them. The operating system performs this interleaving with a mechanism known as **context switching.**

​	At any point in time, a uniprocessor system can only execute the code for a single process. When the operating system decides to transfer control from the current process to some new process, it performs a context switch by saving the context of the current process, restoring the context of the new process, and then passing control to the new process. The new process picks up exactly where it left off.

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503155549297.png" alt="image-20220503155549297" style="zoom:67%;" />

​	As Figure 1.12 indicates, the transition from one process to another is managed by the operating system kernel. The kernel is the portion of the operating system code that is always resident in memory. When an application program requires some action by the operating system, such as to read or write a file, it executes a special system call instruction, transferring control to the kernel. The kernel then performs the requested operation and returns back to the application program. **Note that the kernel is not a separate process. Instead, it is a collection of code and data structures that the system uses to manage all the processes.**

### 7.2 Threads

​	Although we normally think of a process as having a single control flflow, in modern systems a process can actually consist of multiple execution units, called *threads*, each running in the context of the process and sharing the same code and global data.



### 7.3 Virtual Memory

Virtual memory is an abstraction that provides each process with the illusion that it has exclusive use of the main memory.

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503160724473.png" alt="image-20220503160724473" style="zoom:50%;" />

1. **Program code and data.**Code begins at the same fixed address for all processes, followed by data locations that correspond to global C variables. The code and data areas are initialized directly from the contents of an executable object file—in our case, the hello executable.
2. **Heap.**The code and data areas are followed immediately by the run-time *heap*. Unlike the code and data areas, which are fixed in size once the process begins running, the heap expands and contracts dynamically at run time as a result of calls to C standard library routines such as *malloc* and *free**
3. **Shared libraries.**Near the middle of the address space is an area that holds the code and data for *shared libraries* such as the C standard library and the math library. The notion of a shared library is a powerful but somewhat diffificult concept. 
4. **Stack.** At the top of the user’s virtual address space is the *user stack* that the compiler uses to implement function calls. Like the heap, the user stack expands and contracts dynamically during the execution of the program. In particular, each time we call a function, the stack grows. Each time we return from a function, it contracts.
5. **Kernel virtual memory.**The top region of the address space is reserved for the kernel. Application programs are not allowed to read or write the contents of this area or to directly call functions defined in the kernel code. Instead, they must invoke the kernel to perform these operations

### 7.4 Files

​	A file is a sequence of bytes, nothing more and nothing less. Every I/O device, including disks, keyboards, displays, and even networks, is modeled as a file. All input and output in the system is performed by reading and writing files, using a small set of system calls known as Unix I/O. This simple and elegant notion of a file is nonetheless very powerful because it provides applications with a uniform view of all the varied I/O devices that might be contained in the system. For example, application programmers who manipulate the contents of a disk file are blissfully unaware of the specific disk technology. Further, the same program will run on different systems that use different disk technologies. 

## 8.Systems Communicate with Other Systems Using Networks

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220503163244041.png" alt="image-20220503163244041" style="zoom:50%;" />

​	Network can be viewed as just another I/O device, as shown in Figure 1.14. When the system copies a sequence of bytes from main memory to the network adapter, the data flow across the network to another machine, instead of, say, to a local disk drive. Similarly, the system can read data sent from other machines and copy these data to its main memory



## 9.Amdahl’s Law

https://zhuanlan.zhihu.com/p/48022905

​	Gene Amdahl, one of the early pioneers in computing, made a simple but insightful observation about the effectiveness of improving the performance of one part of a system. This observation has come to be known as Amdahl’s law. The main
idea is that when we speed up one part of a system, the effect on the overall system performance depends on both **how significant this part was** and **how much it sped up**. Consider a system in which executing some application requires time
Told. Suppose some part of the system requires a fraction α of this time, and that we improve its performance by a factor of k. That is, the component originally required time αTold, and it now requires time (αTold)/k. The overall execution time would thus be:

```mathematica
Tnew = (1 − α)Told + (α/k)Told
		 = [(1 − α) + α/k]Told
From this, we can compute the speedup 
S = Told/Tnew 
  = 1 /((1 − α) + α/k))
```