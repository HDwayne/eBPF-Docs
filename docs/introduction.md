# What is eBPF ? 

eBPF, or extended Berkeley Packet Filter, is a technology that enables secure execution of code within the kernel of an operating system (Windows or Linux). Initially developed for network packet filtering, BPF evolved into eBPF to become a more general technology capable of injecting and executing more complex and diversified code.



# Operation

## eBPF program structure

In most cases, the use of eBPF technology revolves around two types of programs:

### User-space program :
This is the code that runs in user space (outside the kernel). This program is responsible for managing and interacting with the eBPF code injected into the kernel. It can be used to inject/remove eBPF code, retrieve results via eBPF maps, etc.

This part of the code is typically written in C, but APIs that allow the use of higher-level languages such as BCC (BPF Compile Collection, which uses Python) or eBPF-GO (which uses Go) can also be utilized. In C, some libraries exist to facilitate the writing of this part of the program, such as libbpf.

### Kernel space program :

This is the actual BPF code injected into the kernel. This code is often referred to as "eBPF program" or "kernel space program." It can be dynamically injected into the kernel and executed in response to certain events, such as the arrival of network packets or the invocation of a system command by a process.

In addition to being necessarily written in C, this part of the code is extremely constrained due to its execution within the kernel. For example:

- These programs cannot use uninitialized variables or access memory beyond limits.
- These programs must have a size allowing them to be injected into the kernel.
- They must have finite complexity.
- They must not be able to block in any way (infinite loops, etc.).
- The process injecting the eBPF code into the kernel must have the necessary privileges.

These constraints are ensured to be respected by the eBPF verifier (see Compilation and Verification section).


## Compilation and verification


The eBPF code requires certain operations before being injected into the kernel. The first step is to compile the file into bytecode (ELF format) because this is the file type expected by the kernel.

![im1](https://ebpf.io/static/a7160cd231b062b321f2a479a4d0848f/9180b/clang.png "compilation d'un programme eBPF en fichier ELF")

Clang and GCC (since version 10) support the compilation of eBPF files.

Subsequently, the ELF file undergoes a verifier that ensures the program will run correctly within the kernel. The purpose is to guarantee the absence of potential crashes or program blockages during its execution and to ensure that there are no security vulnerabilities. Without this verification, running an eBPF program would be extremely risky.

![im2](https://ebpf.io/static/7eec5ccd8f6fbaf055256da4910acd5a/b5f15/loader.png "Processus d'exécution d'un programme eBPF: de la vérification à l'injection au sein du kernel")


The ELF file then goes through a Just-in-Time (JIT) compiler that transforms it into Assembly language instructions (specific to the machine architecture on which the file is located). Finally, the program is attached to the system event with which it has been associated.



## eBPF maps 


eBPF maps are data structures used by eBPF programs. They primarily serve to store and share data between user space and kernel space, as well as between different instances of eBPF programs (other eBPF programs can access them, as well as programs outside the kernel through system calls). Since there can be concurrent access, map operations are atomic to preserve data coherence.

Information is stored persistently, allowing access at any time.

## Events 

Each eBPF program must be attached to an "event" within the kernel. When this "event" occurs, the eBPF program is executed. There are numerous events of different types:

- eBPF programs can be attached to events related to the arrival and processing of network packets using the eXpress Data Path (XDP) subsystem. They can also be associated with events related to network traffic management.

- eBPF programs can be attached to specific locations in the kernel (tracepoints) to collect real-time information, allowing detailed analysis of system execution.

- eBPF programs can be attached to entry or exit points of kernel functions, enabling the creation of system probes (Kprobes) for debugging, monitoring, and other tasks.

- eBPF programs can be attached to the Linux Security Module (LSM) and influence the behavior of the kernel.

- and much more...










