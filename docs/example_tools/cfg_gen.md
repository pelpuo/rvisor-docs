# Control Flow Graph Generation

## Granularity
Basic Block Level

## Description
Generates an annotated control flow graph for a target binary.

This in an inbuilt feature within R-Visor. Whenever a basic block terminates and navigates back to the dispatcher, this routine uses information about the current basic block and target basic block to create an entry for a dotfile which can be used to generate the control flow graph when the program terminates.

## Code Reference
```c++
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <map>

#include "../src/system_calls.h"
#include "../src/decode.h"
#include "../src/rail.h"
#include "../src/regfile.h"
#include "../src/printUtils.h"

using namespace rail;

void exitRoutine(uint64_t *regfile){
    asm volatile ("csrr %0, instret" : "=r"(end_instr));
    asm volatile ("rdcycle %0" : "=r"(end_cycle));

    rail::outfile << "Total elapsed cycles: " << end_cycle-start_cycle << endl;
    rail::outfile << "Total instructions executed: " << end_instr-start_instr << endl;
}

int main(int argc, char** argv) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.setLoggingFile("cfg_gen_logs");
    railer.addCFGGeneration("cfg_gen.dot");

    railer.runInstrument();
}
```