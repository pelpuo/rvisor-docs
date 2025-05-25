# Power Analysis

## Granularity
Instruction Type

## Description
Outputs the per instruction power consumption of a target binary.

The routine is called at every instruction. During the routine, the power consumption for each instruction is checked, based on a predetermined map. The power is output to a file and a waveform can be generated with the resulting output.

## Code Reference
```c++
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <map>
#include <unordered_map>

#include "../src/system_calls.h"
#include "../src/decode.h"
#include "../src/rail.h"
#include "../src/regfile.h"
#include "../src/printUtils.h"

using namespace rail;

std::unordered_map<InstName, double> instCurrentMap = {
        {ADD_INST, 1749.85},
        {SUB_INST, 1744.99},
        {SLL_INST, 1747.28},
        {SLT_INST, 1750.97},
        {SLTU_INST, 1749.58},
        {XOR_INST, 1745.74},
        {SRL_INST, 1744.38},
        {SRA_INST, 1747.34},
        {OR_INST, 1744.44},
        {AND_INST, 1749.85},
        {MUL_INST, 1757.36},
        {MULH_INST, 1757.10},
        {MULHSU_INST, 1758.07},
        {MULU_INST, 1757.98},
        {DIV_INST, 2942.86},
        {DIVU_INST, 1476.47},
        {REM_INST, 2932.05},
        {REMU_INST, 1465.62},
        {ADDI_INST, 1750.45},
        {SLLI_INST, 1926.32},
        {SLTI_INST, 1749.31},
        {SLTIU_INST, 1750.24},
        {XORI_INST, 1745.51},
        {SRLI_INST, 1972.56},
        {SRAI_INST, 1969.52},
        {ORI_INST, 1745.47},
        {ANDI_INST, 1748.59},
        {LUI_INST, 1749.08},
        {AUIPC_INST, 1751.49},
        {LB_INST, 2453.45},
        {LH_INST, 2465.97},
        {LW_INST, 2434.12},
        {LBU_INST, 2450.49},
        {LHU_INST, 2458.88},
        {SB_INST, 1838.31},
        {SH_INST, 2896.19},
        {SW_INST, 2727.06},
        {BEQ_INST, 2904.98},
        {BNE_INST, 2924.13},
        {BLT_INST, 2936.27},
        {BGE_INST, 2914.36},
        {BLTU_INST, 2933.06},
        {BGEU_INST, 2920.13},
        {JAL_INST, 2919.94},
        {JALR_INST, 2919.94},
        {MULW_INST, 1757.36},
        {DIVW_INST, 2942.86},
        {DIVUW_INST, 1476.47},
        {REMW_INST, 2932.05},
        {REMUW_INST, 1465.62},
        {ADDIW_INST, 1750.45},
        {SLLIW_INST, 1926.32},
        {SRLIW_INST, 1972.56},
        {SRAIW_INST, 1969.52},
        {LWU_INST, 2434.12},
        {LD_INST, 2434.12},
        {SD_INST, 2727.06},
        {FENCE_INST, 0.0},
        {ECALL_INST, 2942.86 *10},
        {EBREAK_INST, 2942.86 *10},
        {UNDEF_INST, 0.0}
};

double powerVal = 0;
void powerAnalysisRoutine(RvInst instruction, uint64_t *regfile){
    if(instCurrentMap.count(instruction.name)){
        powerVal = instCurrentMap[instruction.name];
        outfile << powerVal << std::endl;
    }
}

int main(int argc, char** argv, char** envp) {

    if(argc < 2){
        cout << "Please provide a target binary" << std::endl;
        exit(1);
    }

    rail::Rail railer;
    railer.setTarget(argv[1]);
    railer.registerArgs(argc-1, &argv[1], &envp[0]);
    railer.setLoggingFile("power_analysis_msd_logs");

    railer.addInstrumentInstAll(powerAnalysisRoutine, InsertPoint::POST);

    railer.runInstrument();
}
```