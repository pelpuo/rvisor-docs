# Trace Linking

## Description
During binary execution, R-Visor regains control under the following scenarios: 
* The next basic block is not already in the cache, 
* The dispatcher cannot statically determine the target of a jump instruction during BB allocation, or 
* R-Visor needs to execute an instrumentation routine.

 In the absence of these conditions, the basic blocks can be connected together, minimizing the overheads associated with the context switches. We use this ideology for our version of trace linking. Separate algorithms are used to achieve trace linking for direct jumps and branches. However, these algorithms are abstracted from the user to simplify the R-Visor API.

 ## Use Case
 To measure the impact of trace linking, we will execute the **no_instruemtation** routine with and without trace linking enabled. This will be done by measuring the cycles and instruction count, through RISC-V's Hardware Performance Counters (HPCs). 
 
 ### Measuring metrics without trace linking
 By default, trace linking is disabled so we can enable the recording of metrics in R-Visor by modifying the **CMakeLists.txt** file.

```cmake

# CMakeLists.txt

# Insert the -DMETRICS flag to this part of the code
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -DMETRICS -march=rv64imafd -mabi=lp64d -mno-relax")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -DMETRICS -march=rv64imafd -mabi=lp64d -mno-relax")


 ```

Ensure that the executable being built is the **no_instrumentation** routine.

```cmake
# CMakeLists.txt

add_executable(no_instrumentation ${ROUTINESDIR}/no_instrumentation.cpp ${HEADER_FILES})
```

Build the binary

```bash
cmake .
make
```

Running `make` would build a binary called `no_instrumentation`.

Execute this binary while instrumenting the `stream` benchmark to generate the metrics.

```bash
./no_instrumentation benchmarks/stream
```

The metrics would be stored in `no_instrumentation.txt`.

```
Total elapsed cycles: 3623305178
Total instructions executed: 3623305172
```

### Measuring metrics with trace linking
To measure the impact of trace linking, we will duplicate `no_instrumentation.cpp` in `/routines` and rename the copy as `no_instrumentation_tl.cpp`.

We will edit this file to enable trace linking
```c++
// no_instrumentation_tl.cpp

int main(int argc, char** argv, char** envp) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }


    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.registerArgs(argc-1, &argv[1], &envp[0]);

    // Change the name of the logging file so that we do not 
    // overwrite the old metrics
    railer.setLoggingFile("no_instrumentation_tl_logs");
    
    // Insert this line to enable trace linking
    railer.enableTraceLinking();

    #ifdef METRICS
        railer.setExitRoutine(exitRoutine);
        asm volatile ("rdcycle %0" : "=r"(start_cycle));
        asm volatile ("csrr %0, instret" : "=r"(start_instr));
    #endif
    
    railer.runInstrument();
}

```

Modify the `CMakeLists.txt` so that we can build the `no_instrumentation_tl.cpp` file.

```cmake
# CMakeLists.txt

# Add this line
add_executable(no_instrumentation_tl ${ROUTINESDIR}/no_instrumentation_tl.cpp ${HEADER_FILES})
```

Build the binary

```bash
make
```

Running `make` would build a binary called `no_instrumentation_tl`.

Once again execute this binary while instrumenting the `stream` benchmark to generate the metrics.

```bash
./no_instrumentation_tl benchmarks/stream
```

The metrics would be stored in `no_instrumentation_tl.txt`.

```
Total elapsed cycles: 505284725
Total instructions executed: 505284719
```

After comparing the results, it is evident that there is a very large improvement in the cycle and instruction count (~85%).

## Conclusion
Although we have demonstrated that trace linking improves the computation performance of a binary being instrumented, we only demonstrated this without any instrumentation routines present. This is because, by default, instrumentation routines require a context switch to R-Visor. Thus if we need to run routines at the basic block or instruction granularity, the benefits of using trace linking would be lost. To overcome this, we implemented another optimization called [routine inlining](./routine_inlining).