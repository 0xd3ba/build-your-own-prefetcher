# Preliminaries - Getting Started

The first question that you might have had is, "*What exactly is [ChampSim](https://github.com/ChampSim/ChampSim)* ?" As mentioned previously,
> ChampSim is a *trace-based* simulator for conducting micro-architectural research 

Loosely speaking, it is a simulator (with tweakable parameters), which takes a *trace* and spits out some numbers
quantifying how the system performed on that particular trace. Naturally, you might now wonder "*What exactly is a trace* ?"
There is no fixed definition on what exactly a trace is, but you can think of it as

> Trace-files, or simply put, traces, are special files which contain the dynamics of a particular program within some specific
> region of interest, in temporal order.
> It may include the list of instruction-pointers (IPs), the list of addresses to which loads/stores were issued etc.

What exactly goes inside a trace, depends on the needs and who is writing the *tracer*, using [Intel's PIN](https://www.intel.com/content/www/us/en/developer/articles/tool/pin-a-dynamic-binary-instrumentation-tool.html)
(which is a completely different problem, not discussed here). Thankfully, ChampSim already has a tracer written for it,
which can be used to generate traces, *in the format **specific** to ChampSim*. 
Luckily, we don't even have to even deal with creating ChampSim traces for the most part, since the
ChampSim traces of popular *benchmark-suites*, for ex. [SPEC2017](https://www.spec.org/cpu2017/), are already made publicly 
available for research.

## Hands-on with ChampSim
In this section, we'll get our hands dirty with ChampSim by checking
how a system with *IPCP* [1] prefetcher performs against a system 
with *no* prefetching on the following trace: `602.gcc_s-2226B.champsimtrace.xz`. 
Note that you don't have to read the paper before/after trying out this experiment, but if you're really interested, I recommend
you start off with the workshop paper [2], which is only 4 pages, before reading the full paper [1].

**NOTE:** Ensure that you have `g++` (v7.0 or later) and `make` installed in your system before building ChampSim

Follow the given steps:

1. Clone [ChampSim](https://github.com/ChampSim/ChampSim) repository.
You'll notice a lot of directories inside `ChampSim/`. The `prefetcher/` directory stores the **C++** source code (don't get confused
by the weird file extensions) for the prefetchers, which is of interest. The naming convention is as follows

    | Extension | Type |
    |:------:|:-----|
    |`*.l1i_pref` | L1-I Cache Prefetcher |
    |`*.l1d_pref` | L1-D Cache Prefetcher |
    |`*.l2c_pref` | L2 Cache Prefetcher |
    |`*.llc_pref` | L3 Cache (or LLC) Prefetcher |

2. Download the prefetcher files from [IPCP](https://github.com/car3s/IPCP_ISCA2020) 's repository and place them inside
ChampSim's `prefetcher/` directory.

3. Now let's build ChampSim. We'll be building two binaries for single-core CPU
    * System with *no* prefetchers
    * System with *IPCP* prefetchers at LI-D and L2C

4. While being inside `ChampSim/` directory (in the shell), execute the following commands

    * `./build_champsim.sh bimodal no no no no lru 1`                                                                                            
    * `./build_champsim.sh bimodal no ipcp ipcp no lru 1`
 
    Each command will take some time (few seconds generally) to finish executing. As for understanding the arguments to
    `build_champsim.sh` script, please refer to ChampSim's repository. It's explained there and is pretty straight-forward.
    
5. Assuming you did everything right till now, the builds will finish successfully and a new directory `bin/` will be created
containing the following two files:

    ```
    ChampSim/
      ├── ...
      ├── bin/
      │     ├── bimodal-no-no-no-no-lru-1core
      │     └── bimodal-no-ipcp-ipcp-no-lru-1core
      ├── ...
   
    ```
      
6. Create a new directory `dpc3_traces/` within `ChampSim/`. Download the following trace
[`602.gcc_s-2226B.champsimtrace.xz`](https://hpca23.cse.tamu.edu/champsim-traces/speccpu/index.html) (credits to Prof. Daniel Jimenez from Texas A&M University) 
and place it inside the `dpc3_traces/` directory. The directory structure should look something like this

    ```
    ChampSim/
      ├── ...
      ├── bin/
      │     ├── bimodal-no-no-no-no-lru-1core
      │     └── bimodal-no-ipcp-ipcp-no-lru-1core
      ├── dpc3_traces/
      │     └── 602.gcc_s-2226B.champsimtrace.xz
      ├── ...
   
    ```  

7. Now we are finally ready to start the evaluation. We'll run each binary for a *warm-up* of 10-million instructions and
a *simulation* of 10-million instructions (higher values are typically preferred, but for our case the above would suffice).
Execute the following commands:

    * `./run_champsim.sh bimodal-no-no-no-no-lru-1core 10 10 602.gcc_s-2226B.champsimtrace.xz`                                                                                           
    * `./run_champsim.sh bimodal-no-ipcp-ipcp-no-lru-1core 10 10 602.gcc_s-2226B.champsimtrace.xz`
  
    It will take some time to finish the simulation (typically 1-2 min each). Similar to before, please refer to ChampSim's
    repository to understand the arguments to `run_champsim.sh` script. It is pretty straight-forward too.

8. Once simulation completes, a new directory `results_10M/` will be created containing the corresponding simulation results as `*.txt` files.
Don't worry about the contents of the files yet. For now, simply focus on the IPC (Instructions-per-clock), which is present as following in each result

    ```
    ... (content)
   
    Region of Interest Statistics
   
    CPU 0 cumulative IPC: xxxxxx
    
    ... (more content)
    ```

9. Is the IPC with IPCP prefetcher higher or lower ? 
    * By how much ? 
    * Did prefetching help ?

## Conclusion
Congratulations! You have reached the end of the preliminaries, assuming you indeed tried out the above. Simulating with
ChampSim is that easy, unless you get SegFaults (hehe). Assuming you have written a prefetcher, you basically have to repeat the
above steps with your prefetcher (at the corresponding levels) instead of IPCP, to get the results. You have to repeat this
with every prefetcher that you want to compare your prefetcher against (discussed later) before making conclusions about your implementation.

To get the feel, try out the following exercises.

## Exercises
1. Try a warm-up of 1-million and simulation of 1-million instructions each. What change in IPC do you observe ? 
    * What about 30-million instructions warm-up and 50-million instructions for simulation ? (It may take 15-30 min each, in this case)
2. Previously we were using IPCP at both L1-D and L2 caches. What would be the change in IPC if **no prefetcher is used at L2 cache** ? 
(Use 10-million instructions for both warm-up and simulation, similiar to before)

## References
[1] Samuel Pakalapati and Biswabandan Panda, [“Bouquet of Instruction Pointers: Instruction Pointer Classifier-based Spatial Hardware Prefetching”](https://www.cse.iitk.ac.in/users/biswap/IPCP_ISCA20.pdf), 
In Proceedings of the 47th International Symposium on Computer Architecture, Valencia, Spain [virtual one], 2020

[2] Samuel Pakalapati and Biswabandan Panda, [“Bouquet of Instruction Pointers: Instruction Pointer Classifier-based Hardware Prefetching”](https://www.cse.iitk.ac.in/users/biswap/DPC3.pdf)
(Winner of [DPC3@ISCA 2019](https://dpc3.compas.cs.stonybrook.edu) Championship)