# Instruction Counting
The tool which would be built in this section counts all executed instructions in a target binary. This tool will be created in `example/inst_counting.cpp` within the R-Visor project directory.

## Creating the Routines
The sub tasks for the creation of this tool are:
1. Creating a variable to keep track of the number of instructions
2. Increment the variable every time an instruction is executed
3. Print out the total number of instructions encountered at the end of the program

To keep track of instructions we can create a variable

```c++
int numInstructions;
```

Next, we need to create an instruction level routine which would increment the value of `numInstructions`. The routine is created as a function within the file, and the function must include specific arguments:
* a parameter of type `RvInst`. This is a struct which contains information about the instruction.
* A parameter of type `uint64_t *`. This is a pointer to an array containing all the register values at the time of invocation of the routine.

Within the function body, we can simply increment `numInstructions`.

```c++
void instCounting(rail::RvInst railInst, uint64_t *regfile){
    numInstructions++;
}
```

To print the number of instructions at the end, we must use a module level routine. R-Visor specifies the format for a kind of module level routine which executes at the end of the program. The routine contains a single parameter of type `uint64_t *`. Rather than using the `std::cout` to print the output to the terminal, we would be printing the number of instructions to a log file specified by R-Visor. This is the `outfile`.

```c++
void exitRoutine(uint64_t *regfile){
    outfile << "Num Instructions: " << numInstructions << endl;
}
```
## Creating `main()`
Now we must create the main function for the tool. The main function does the following:
1. Initialize our R-Visor object to manage the instrumentation. This is the `railer` object.

2. Register the target binary. Since we will be passing the name of the binary through the command line arguments, we can simply pass `argv[1]` to `railer` using `railer.setTarget(argv[1])`

3. The target binary may also need access to the command line arguments. Thus we also pass them to the binary through `railer` by using `railer.registerArgs(argc-1, &argv[1], &envp[0])`

4. The name of the R-Visor logging file is specified with `railer.setLoggingFile("bb_counting.txt")`

5. Next we must register our routines. To register the instruction level routine, we use `railer.addInstrumentInstAll(BBCounting, rail::InsertPoint::POST)`. The second parameter to this (`rail::InsertPoint::POST`) specifies the [routine placement](./../overview/routine_placement.md). We use `POST` to place our routines after the instruction has executed. Alternatively, we can change the value to `PRE` since our routine is not dependent on the register values.

6. We register the module level routine with `railer.setExitRoutine(exitRoutine)` which executes after the binary has invoked the `exit` syscall.

7. Lastly, we use `railer.runInstrument()` to begin the JIT execution of the binary.


```c++
int main(int argc, char** argv, char**envp) {
    
    numBasicBlocks = 0;

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.registerArgs(argc-1, &argv[1], &envp[0]);
    railer.setLoggingFile("inst_counting.txt");

    railer.addInstrumentInstAll(instCounting, rail::InsertPoint::POST);
    railer.setExitRoutine(exitRoutine);
    
    railer.runInstrument();
}
```

The full code for this routine is listed below:

```c++
#include <stdio.h>
#include <stdlib.h>

#include "../src/decode.h"
#include "../src/rail.h"
#include "../src/regfile.h"


using namespace rail;
int numInstructions;

void exitRoutine(uint64_t *regfile){
    outfile << "Num Instructions: " << numInstructions << endl;
}

void instCounting(rail::RvInst railInst, uint64_t *regfile){
    numInstructions++;
}

int main(int argc, char** argv, char**envp) {
    
    numBasicBlocks = 0;

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.registerArgs(argc-1, &argv[1], &envp[0]);
    railer.setLoggingFile("inst_counting.txt");

    railer.addInstrumentInstAll(instCounting, rail::InsertPoint::POST);
    railer.setExitRoutine(exitRoutine);
    
    railer.runInstrument();
}
```

## Building our Tool
To build our tool we must modify the `CMakeLists.txt` file.

```cmake
# CMakeLists.txt

# Add this line
add_executable(inst_counting ${EXAMPLESDIR}/inst_counting.cpp ${HEADER_FILES})
```

Next we can build our tool

```
cmake .
make
```

This will create a binary called `inst_counting`. We can execute this on a RISC-V binary. For this example, we 

```
./inst_counting benchmarks/fib
```

After the tool executes, the results are stored in `inst_counting.txt`

```
Num Instructions: 6692
```