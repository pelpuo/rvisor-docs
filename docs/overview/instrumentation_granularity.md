# Instrumentation Granularity
Instrumentation routines in R-Visor are executed periodically at a frequency determined by the user. The frequency or granularity of instrumentation routines can be one of the following:
* Module  
* Basic Block 
* Instruction 
* Instruction Type
* Instruction Pseudo Type

## Module Granularity
The **module level** routines execute once per program. A typical use case for this is an exit routine which only executes at the end of a program to print out collected data or perform some other function. 

## Basic Block Granularity
The **basic block level** routines execute whenever there is a control flow change within the instrumented binary. Since, by default, R-Visor interrupts the program execution when control flow instructions are encountered, there are no additional context switches required to execute **basic block level** routines. When instrumenting at this granularity, users have access to basic block metadata collected by R-Visor. The metadata is of the following form:

```c++
typedef enum{
    INDIRECT_BRANCH,
    DIRECT_BRANCH,
    FUNCTION_CALL,
    FUNCTION_RETURN,
    SYSCALL,
    SEGMENTED
}BasicBlockType;

typedef struct {
    uint64_t firstAddr; // First address from binary
    uint64_t lastAddr; // Last address from binary
    uint64_t startLocationInCache; // memoryIndex where block begins in cache
    uint64_t endLocationInCache; // memoryIndex where block ends in cache
    int numInstructions;
    bool takenBlock; // Control switched here from a taken branch from a previous block 
    BasicBlockType type; // Type of basic block (Dependent on last Instruction)
    uint32_t startInst; // Bytes for last instruction
    uint32_t terminalInst; // Bytes for last instruction
    uint64_t basicBlockAddress; // If block is segmented then this stores the address of the first inst
}RailBasicBlock;
```

Example routines implemented at this granularity are [BBProfile](./../example_tools/bb_profile) and [BBFrequency](./../example_tools/bb_frequency)

## Instruction Level Granularity
Routines can be placed at **instruction level** granularity when a user requires statistics for each individual instruction executed in the target binary. By default, a context switch would need to be invoked after every instruction for this instrumentation granularity, unless [inline routines](./../optimizations/routine_inlining) are used. As such, this is the most computationally expensive type of routine. At this granularity, users have access to the decoded instruction as a struct created by R-Visor. This has the following form:

```c++
typedef struct {
    int opcode;
    int rd;
    int funct3;
    int rs1;
    int rs2;
    int funct7;
    int imm;
    int address;

    InstType type;
    InstName name;
} RvInst;
```

## Instruction Type Granularity
The RISC-V instruction set defines 6 distinct instruction types (for uncompressed instructions) namely:

* R Type
* I Type
* S Type
* U Type
* B Type
* J Type

Based on these, a modified version of the instruction level granularity was created in R-Visor, in order for users to instrument at just one of these types of instructions, rather than for the whole program. 

## Instruction Pseudotype Granularity
In addition to the standard RISC-V instruction types, R-Visor also allows users to instrument based on custom defined types. The base R-Visor includes two pseudotypes namely:
* Memory Access Type: For Load and Store instructions and
* System Type: For `ECALL` and `EBREAK` instructions

The R-Visor Codebase can also be extended to include additional pseudotypes based on the user's required functionality. Example routines at this granularity are [MemAccessProfile](./../example_tools/mem_access_prof) and [SyscallTrace](./../example_tools/syscall_trace)