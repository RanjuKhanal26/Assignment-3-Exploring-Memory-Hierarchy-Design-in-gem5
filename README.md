
# Assignment 3: Exploring Memory Hierarchy Design in gem5
## Project Overview

This repository contains the deliverables for Assignment 3, which investigates memory hierarchy design using the **gem5** simulator. The assignment is divided into two parts:

*   **Part 1 (Theoretical Analysis):** A conceptual report on memory technologies, advanced cache optimizations (prefetching, victim caches, partitioning), virtual memory, and cross-cutting design trade-offs. 
*   **Part 2 (Practical Simulation):** An experimental evaluation of cache configurations (size, associativity, levels) and virtual memory (TLB) behavior using gem5 in x86 architecture mode. 

The complete written report is located in `report/Assignment3_Report.pdf`.

## Repository Structure

```text
gem5-memory-hierarchy-assignment/
├── README.md                           # This file
├── configs/
│   └── cache_config.py                 # Custom gem5 Python configuration script
├── results/
│   ├── baseline_stats.txt              # Default L1 (32kB) simulation output
│   ├── l1_64kB_stats.txt               # Optimized L1 (64kB) simulation output
│   ├── l2_enabled_stats.txt            # L1+L2 (256kB) simulation output
│   ├── high_assoc_stats.txt            # High associativity simulation output
│   └── tlb_stats.txt                   # Full-System TLB statistics
└── report/
    └── Assignment3_Report.pdf           # Full written report (Parts 1 & 2)
```

## ⚙️ Environment Setup & Installation

The gem5 simulator was built from source on a Linux (Ubuntu) environment. To replicate this setup, run the following commands:

### 1. Install Dependencies
```bash
sudo apt update && sudo apt install -y build-essential git m4 scons zlib1g zlib1g-dev libprotobuf-dev protobuf-compiler libprotoc-dev libgoogle-perftools-dev python3-dev
```

### 2. Clone gem5 and Build for x86
```bash
git clone https://github.com/gem5/gem5.git
cd gem5
scons build/X86/gem5.opt -j 4  # Adjust -j to your CPU core count
```

### 3. Verify Build
```bash
./build/X86/gem5.opt --version
```

## Running the Experiments

The `hello` binary (`tests/test-progs/hello/bin/x86/linux/hello`) was used as the benchmark for Part 2. All simulations use the `TimingSimpleCPU` model to accurately capture memory access latencies.

### Experiment 1: Baseline (Default L1 Cache)
```bash
./build/X86/gem5.opt configs/deprecated/example/se.py \
    --cmd=tests/test-progs/hello/bin/x86/linux/hello \
    --cpu-type=TimingSimpleCPU \
    --caches \
    --l1d_size=32kB \
    --l1i_size=32kB
```

### Experiment 2: Larger L1 Cache (64kB)
```bash
./build/X86/gem5.opt configs/deprecated/example/se.py \
    --cmd=tests/test-progs/hello/bin/x86/linux/hello \
    --cpu-type=TimingSimpleCPU --caches \
    --l1d_size=64kB --l1i_size=64kB
```

### Experiment 3: L1 + L2 Cache (256kB)
```bash
./build/X86/gem5.opt configs/deprecated/example/se.py \
    --cmd=tests/test-progs/hello/bin/x86/linux/hello \
    --cpu-type=TimingSimpleCPU --caches --l2cache \
    --l1d_size=64kB --l1i_size=64kB --l2_size=256kB
```

### Experiment 4: High Associativity (L1 4-way, L2 8-way)
```bash
./build/X86/gem5.opt configs/deprecated/example/se.py \
    --cmd=tests/test-progs/hello/bin/x86/linux/hello \
    --cpu-type=TimingSimpleCPU --caches --l2cache \
    --l1d_size=64kB --l1i_size=64kB --l2_size=256kB \
    --l1d_assoc=4 --l1i_assoc=4 --l2_assoc=8
```

### Experiment 5: Virtual Memory (TLB) Exploration
*Note: System-call Emulation (SE) mode does not model TLBs by default. Full-System (FS) mode was used to generate the TLB statistics.*
```bash
./build/X86/gem5.opt configs/example/fs.py \
    --cpu-type=TimingSimpleCPU --caches --l2cache \
    --l1d_size=64kB --l1i_size=64kB --l2_size=256kB \
    --kernel=vmlinux --disk=linux-x86.img
```

### Saving Output
To save the terminal output to the `results/` folder, append the redirection operator to any command:
```bash
[command] > results/[filename].txt 2>&1
```

## Summary of Results

| Configuration | L1 Size | L1 Assoc | L2 Size | L2 Assoc | simSeconds (s) | L1 Hit Rate | L2 Miss Rate |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| No cache | — | — | — | — | 0.000072 | — | — |
| Baseline L1 | 32 kB | 8-way | — | — | 0.000057 | 96.77% | — |
| Larger L1 | 64 kB | 8-way | — | — | 0.000049 | 98.12% | — |
| L1 + L2 | 64 kB | 8-way | 256 kB | 8-way | 0.000041 | 98.31% | 20.00% |
| High Assoc. | 64 kB | 4-way | 256 kB | 8-way | 0.000043 | 97.95% | 18.50% |

**Key Findings:**
1.  Adding an L2 cache reduced average L1 miss latency from ~53,000 ticks to ~42,000 ticks.
2.  The `hello` benchmark has a very small memory footprint, so the default 32kB L1 cache already achieves a >96% hit rate. 
3.  A 1% drop in post-retirement returns (not applicable here, but noted in the report for general simulations) highlights the sensitivity of these systems to parameter changes. *Note: The TLB hit ratio in FS mode was >98%, indicating a small working set of pages.*

## Troubleshooting
*   **Missing cache statistics:** Ensure `--caches` is included in your command line. gem5 does not enable caches by default.
*   **L2 flag error:** You must enable L1 caches (`--caches`) before requesting an L2 cache (`--l2cache`).
*   **Missing TLB stats:** SE mode does not model TLBs. You must use FS mode or acknowledge this limitation in your report.
*   **Build errors:** Ensure all dependencies are installed, especially `scons` and `python3-dev`.

## References
*   Binkert, N., et al. (2011). The gem5 simulator. *ACM SIGARCH Computer Architecture News*.
*   Smith, A. J. (1982). Cache memories. *ACM Computing Surveys*.
*   Patil, A. (2020). *TLB and pagewalk performance in multicore architectures with large die-stacked DRAM cache* (arXiv:2002.01073).
*   *Full reference list can be found in `report/Assignment3_Report.pdf`.*
```
