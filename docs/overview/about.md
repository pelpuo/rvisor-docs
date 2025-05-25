# R-Visor Overview
R-Visor is a dynamic binary analysis and instrumentation library developed by the Secure, Trusted and Assured Microelectronics ([STAM](https://stamcenter.asu.edu/)) center at Arizona State University ([ASU](https://www.asu.edu/)). R-Visor was developed to address the problem of extensibility in Dynamic Binary Instrumentation (DBI) tools. Open ISAs such as RISC-V are constantly changing and introducing new instructions. Thus, DBI tools built for such ISAs must easily be able to integrate support for new instructions. 

We achieve this in R-Visor by leveraging the ArchVisor [Domain Specific Language](https://en.wikipedia.org/wiki/Domain-specific_language#:~:text=A%20domain%2Dspecific%20language%20is,solve%20problems%20in%20many%20domains.) (DSL). Archvisor is a declarative DSL that allows users to include specifications for new ISA extensions in a concise format. The ArchVisor source code is automatically integrated into R-Visor at compile time, allowing users to build instrumentation tools for new extensions

R-Visor uses Just-In-Time execution to execute target application binaries in a sandboxed environment, while running user defined instrumentation routines. Our tool allows for instrumentation of applications compiled in the Executable And Linkable Format (ELF), while maintaining instruction transparency and equivalent control flow to the original binary. R-Visor contains an instrumentation API which enables users to write custom instrumentation tools for a wide variety of purposes such as optimization, program analysis, binary patching and instruction emulation etc. 

We provide initial support for the RISC-V ISA, specifically the rv64gc toolchain. However, thanks to ArchVisor, the tool can be easily extended to support new RISC-V extensions. 

## Instrumenting With R-Visor
R-Visor supports instrumentation at various levels of [granularity](./../overview/instrumentation_granularity), such as

* Module (The entire program)
* Basic Block (A straight line sequence of instructions ending in a control flow change)
* Instruction 
* Instruction Type
* Instruction Pseudo Type

At any of these levels, R-Visor also allows users to insert routines at either a [PRE or POST](./routine_placement) position, depending on the user's intended functionality. R-Visor also contains a number of example instrumentation tools for users who are getting started with the tool.

The current list of tools in the release version of R-Visor includes:

* [BBFrequency](./../example_tools/bb_frequency): A tool for determining the frequency of execution for each basic block within a binary.
* [BBProfile](./../example_tools/bb_profile): To study the distribution of the number of instructions across basic blocks in a binary. 
* [CFG Gen](./../example_tools/cfg_gen): A tool to generate the control flow graph of a binary with utility in bug and hot path detection.
* [MemAccessProfile](./../example_tools/mem_access_prof): A tool to record all memory operations executed within a binary.  
* [PowerAnalysis](./../example_tools/mem_access_prof): For generating the power waveform of a binary.
* [SyscallTrace](./../example_tools/syscall_trace): A tool to record the information on all system call (ECALL) instructions executed in a binary.

## Downloading R-Visor
R-Visor is available for usage and modification. The project is open source and available under an open source licence. 

The latest build of R-Visor is available for download on [Github](https://www.github.com/stamcenter/r-visor)