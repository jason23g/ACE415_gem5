# Lab Gem5 Report

***Team 1***
Ioannis-Jason Georgakas	2017030021
Panos Vasileiou			2017030067


## Introduction


## Q1

We begin by examining the `starter_se.py` file in order to find the characteristics and parameters that the `Hello World` program is run, using the ARM minor CPU.

Initially we observe the `cpu_types` dictionary that contains some pre-defined CPU configurations. The "minor" option has an L1 instruction & data cache and a shared L2 cache. The --cpu is the only option we set and as such all the other options go to their default values. These options are the following:

> CPU frequency	: `4GHz`
> #cores		: `1`
> Memory size	: `2GB`
> Memory type	: `DDR3 1600 MHz 8x8 (Dual channel, no ranks)`


After reading the `SimpleSeSystem` class we discover the following information about the simulated system:

> cache line size			: `64 bytes`
> System components voltage	: `3.3V`
> System components clock	: `1GHz`
> CPU voltage				: `1.2V`

The `starter_se.py` file imports some necessary packages (os, m5, argparse, shlex and common) as well as the python file `devices.py` which is located in the same directory as `starter_se.py`. We find the following:

> L1 instruction cache	: `48kB (3-way)`

> L1 data cache			: `32kB (2-way)`
	>> Write buffers	: `16`

> L2 (shared) cache		: `1MB (16-way)`
	>> Write buffers	: `8`


## Q2

By running the following command:
`./build/ARM/gem5.opt configs/example/arm/starter_se.py --cpu=minor tests/test-progs/hello/bin/arm/linux/hello`

and later opening the `config.ini` we observe:

> cache line size		: cache_line_size in [system]
> L1 data cache			: [system.cpu_cluster.cpus.dcache]
> L1 dcache blk size	: [system.cpu_cluster.cpus.dcache.tags]
> L1 instruction cache	: [system.cpu_cluster.cpus.icache]
> L1 icache blk size	: [system.cpu_cluster.cpus.dcache.tags]
> L2 cache				: [system.cpu_cluster.l2]
> CPU voltage			: [system.cpu_cluster.voltage_domain]
> System voltage		: [system.voltage_domain]
> #cores				: numThreads in [system.cpu_cluster.cpus]
> Memory size			: mem_ranges in [system]
> CPU frequency			: clk\_gate\_max / clock, clk\_gate\_max in [system.cpu_cluster.cpus.power_state] and clock in [system.cpu_cluster.clk_domain]
> System frequency		: clk\_gate\_max / clock, clk\_gate\_max in [system.cpu_cluster.cpus.power_state] and clock in [system.clk_domain]


By oberving the file `stats.txt` we find that the number of comitted instructions is **5028** (*simInsts* in ln. 10). In the following line (*simOps* in ln. 11) there is the number of comitted ops, which is **5834**. The latter consists of ops and micro-ops and is larger due to some instructions possibly being pseudo-instructions.


To find the L2 cache accesses we can read *system.cpu_cluster.l2.overallMisses::total*, which is **479**. Adding the L1 instruction & data cache misses and subtracting the MSHR* hits (Miss Status and Handling Register queue) can also gives us the L2 accesses. If a simulation of the CPU/system was not available, the only way to calculate the accesses would be hardware support for the forementioned CPU/system. An example would be Intel's Performance Counter Monitor.

\*MSHR queue: Holds the list of CPU’s outstanding memory requests that require read access to lower memory level

## Q3

In this question we will conduct some benchmarks using the **Minor** and **TimingSimple** CPUs. Minor is an in-order 4-stage pipelined processor model but with configurable data structures and execute behavior. It's intended use is modelling processors with strict in-order behavior. TimingSimple is derived from the Simple processor model, which is a purely functional in-order model that is suited for cases  where a detailed model is not necessary. TimingSimple is a version that uses the most detailed memory accesses.

>> By running the `insertion_sort.c` for 100 numbers for the **Minor** & **TimingSimple** we get the following results:
Exec. Time Minor		: 0.000374s = `0.374ms`
Exec. Time TimingSimple	: 0.000820s = `0.820ms`
Spd Minor over TimingSimple = 0.820/0.374 = `2.19`

This result is expected since TimingSimple stalls on cache and memory accesses and has no branch prediction in contrast with Minor which has a load/store queue and branch prediction.


>> Changing the frequency of both CPUs to 3GHz we get:
Exec. Time Minor		: 0.000164s = `0.164ms`
Exec. Time TimingSimple	: 0.000292s = `0.292ms`

Spd Minor 3GHz over 1GHz = 0.374/0.164 = `2.28`
Spd TimingSimple 3GHz over 1GHz = 0.820/0.292 = `2.80`
Spd Minor over TimingSimple 3GHz = 0.292/0.164 = `1.78`


By changing the frequency of the CPUs from `1GHz` to `3GHz`, we can see that the speedups it produces vary from `2.2` to `2.8`. Also, the performance deficit of the TimingSimple CPU is smaller with the higher frequency (speedup goes from `2.19` to `1.78`).


>> Changing the memory technology from *DDR3_1600_8x8* to *DDR4_2400_8x8*. This change led to the following results:
Exec. Time Minor		: 0.000373s = `0.373ms`
Exec. Time TimingSimple	: 0.000820s = `0.820ms`

Spd Minor DDR4 over DDR3 = 0.374/0.373 = `1`
Spd TimingSimple DDR4 over DDR3 = 0.820/0.820 = `1`
Spd Minor over TimingSimple DDR4 = 0.292/0.164 = `2.2`

Although these results may present the upgrade to DDR4 memory as meaningless, this is not the case in real systems. This is due to the complex or sometimes random memory access patterns that are often observed. Our program has a simple and relatively straightforward access pattern and a small input. The reason for the latter is that large inputs result in long execution times because the insertion sort algorithm is not very good (compared with quicksort/merge-sort) and the program is run on a simulated CPU.


>> Combining the above changes (3GHz freq. & DDR4 memory) the results are the following:
Exec. Time Minor		: 0.000163s = `0.163ms`
Exec. Time TimingSimple	: 0.000292s = `0.292ms`

Spd Minor 3GHZ & DDR4 over original = 0.374/0.163 = `2.29`
Spd TimingSimple 3GHZ & DDR4 over original = 0.820/0.292 = `2.8`
Spd Minor over TimingSimple 3GHz & DDR4 = 0.292/0.163 = `1.79`

These results are similar with the change to the frequency from 1GHz to 3GHz. As discussed earlier the most important reason is the input size of our program which is limited by the overhead of the simulation.


## Commands

**Starter simulation**
MinorCPU with hello (config: *starter_se.py*)
`./build/ARM/gem5.opt -d results/hello_result configs/example/arm/starter_se.py --cpu=minor --cmd=tests/test-progs/hello/bin/arm/linux/hello`

**Comparing MinorCPU & TimingSimpleCPU**
MinorCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/Minor configs/example/se.py --cpu-type=MinorCPU --cpu-clock=1GHz --mem-type=DDR3_1600_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`

TimingSimpleCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/TimingSimple configs/example/se.py --cpu-type=TimingSimpleCPU --cpu-clock=1GHz --mem-type=DDR3_1600_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`


**Chnaging the clock cycle on MinorCPU & TimingSimpleCPU**
MinorCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/Minor_3G configs/example/se.py --cpu-type=MinorCPU --cpu-clock=3GHz --mem-type=DDR3_1600_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`

TimingSimpleCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/TimingSimple_3G configs/example/se.py --cpu-type=TimingSimpleCPU --cpu-clock=3GHz --mem-type=DDR3_1600_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`


**Chnaging the memory on MinorCPU & TimingSimpleCPU**
MinorCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/Minor_DDR4 configs/example/se.py --cpu-type=MinorCPU --cpu-clock=1GHz --mem-type=DDR4_2400_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`

TimingSimpleCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/TimingSimple_DDR4 configs/example/se.py --cpu-type=TimingSimpleCPU --cpu-clock=1GHz --mem-type=DDR4_2400_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`


**Chnaging the frequency and memory on MinorCPU & TimingSimpleCPU**
MinorCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/Minor_3G_DDR4 configs/example/se.py --cpu-type=MinorCPU --cpu-clock=3GHz --mem-type=DDR4_2400_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`

TimingSimpleCPU with insertion_sort & caches (config: *se.py*)
`./build/ARM/gem5.opt -d results/TimingSimple_3G_DDR4 configs/example/se.py --cpu-type=TimingSimpleCPU --cpu-clock=3GHz --mem-type=DDR4_2400_8x8 --caches --cmd=tests/test-progs/insertion_sort/bin/insertion_sort --options='100'`


## Comments

This exercise was a very good introduction to gem5 and after finishing it the complexity and importance of this tool are evident. However, we had some problems during its installation (failed multiple times) due to a small main memory and swap space (8GB & 1GB respectively).
