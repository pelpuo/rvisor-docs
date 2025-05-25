# Basic Block Frequency

## Granularity
Basic Block Level

## Description
Records the frequency of executed basic blocks within the input binary.

The routine uses a map to keep track of each basic block called. Whenever a basic block is executed, the count within the map is incremented.
The exit routine prints out all entries from the map

## Code Reference
```c++
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <map>

#include "src/rail.h"
#include "src/printUtils.h"
#include "src/decode.h"


using namespace rail;

std:: map<int, int> basicBlockFreqs;

#ifdef METRICS
    uint64_t start_cycle, end_cycle;
    uint64_t start_instr, end_instr;    
#endif

void exitRoutine(uint64_t *regfile){
    for (const auto& pair : basicBlockFreqs) {
        rail::outfile << "Basic Block: " << hex << pair.first << ", frequency: " << dec << pair.second << std::endl;
    }
}

void BBFrequency(rail::RailBasicBlock railBB, uint64_t *regfile){
    if(basicBlockFreqs.count(railBB.firstAddr))basicBlockFreqs[railBB.firstAddr]++;
    else basicBlockFreqs[railBB.firstAddr] = 1;
}


int main(int argc, char** argv, char** envp) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.registerArgs(argc-1, &argv[1], &envp[0]);
    railer.setExitRoutine(exitRoutine);
    railer.setLoggingFile("bb_frequency_logs");

    railer.addInstrumentBBAll(BBFrequency, rail::InsertPoint::POST);

    railer.runInstrument();
}
```