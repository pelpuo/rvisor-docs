# System Call Trace

## Granularity
Instruction Pseudo Type

## Description
This routine prints out information on all system calls executed within the target binary.

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

void ecallRoutine(rail::RvInst railInst, uint64_t *regfile){
    outfile << rail::getSystemCallName((rail::SYSTEMCALL)regfile[regs::A7]) << 
    " (0x" << hex << railInst.address <<")" << std::endl;
}

int main(int argc, char** argv) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.setLoggingFile("syscall_trace_msd_logs");

    railer.addInstrumentInstType(rail::SYSTEM_TYPE, ecallRoutine, rail::InsertPoint::POST);

    railer.runInstrument();
}
```