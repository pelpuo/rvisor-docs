# Memory Access Profiling

## Granularity
Instruction Type

## Description
Records all memory operations (Loads/Stores) within a target binary.

The routine is triggered whenever the `MEM_ACCESS_TYPE` pseudotype is encountered within the binary. The routine prints out the full instruction together with the address.

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

PrintInst printer;

void loadStoreRoutine(rail::RvInst railInst, uint64_t *regfile){
    outfile << "0x" << hex << railInst.address << " ";
    printer.printInstruction(railInst);
}

int main(int argc, char** argv) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.setLoggingFile("mem_access_profiling_logs");

    railer.addInstrumentInstType(rail::MEM_ACCESS_TYPE, loadStoreRoutine, rail::InsertPoint::POST);

    railer.runInstrument();
}
```