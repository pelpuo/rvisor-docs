# Basic Block Profile

## Granularity
Basic Block Level

## Description
Collects statistics on the lengths of basic blocks executed.

The routine uses a map to keep track of the length for each basic block called. Whenever a basic block is executed, the length is checked and the corresponding entry within the map is incremented.
The exit routine prints out all entries from the map


# Code Reference
```c++
using namespace rail;

std:: map<int, int> basicBlockLengths;

void exitRoutine(uint64_t *regfile){
    
    for (const auto& pair : basicBlockLengths) {
        rail::outfile << "Key: " << pair.first << ", Value: " << pair.second << std::endl;
    }
}

void BBProfiling(rail::RailBasicBlock railBB, uint64_t *regfile){
    if(basicBlockLengths.count(railBB.numInstructions))basicBlockLengths[railBB.numInstructions]++;
    else basicBlockLengths[railBB.numInstructions] = 1;
}

int main(int argc, char** argv) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.setExitRoutine(exitRoutine);
    railer.setLoggingFile("bb_profiling_logs");
    
    railer.addInstrumentBBAll(BBProfiling, rail::InsertPoint::POST);

    railer.runInstrument();
}
```