# Build Your Own Prefetcher 

*Prefetching*, in the context of CPU caches, is an optimization technique whose objective is to bring 
instructions/data into the CPU caches *before* they are actually required. An effective prefetch *hides* the
entire latency involved in the transfer from the DRAMs, thus increasing the throughput of a system. An ideal prefetcher
thus gives the illusion that the caches are of infinite capacity.
Although majority of the *state-of-the-art* (SOTA) papers on cache prefetchers use [ChampSim](https://github.com/ChampSim/ChampSim),
which is a *trace-based* simulator for micro-architectural research, to evaluate their approaches with other approaches
(also implemented in ChampSim), the **lack of proper documentation** proves it quite difficult for new-comers to get started
with hands-on research with ChampSim.

This repository tries to reduce, if not eliminate, the difficulty by providing a detailed step-by-step guide on how to use ChampSim
to materialize your prefetching ideas and evaluate them by comparing them against other SOTA approaches, whose ChampSim 
implementations are publicly available online. It does so by starting off with getting you used to ChampSim's framework for
compiling and running an existing SOTA prefetcher on some *SPEC2017 benchmarks* (If you don't know what this is yet, don't worry) 
before moving on to implementing your idea and then evaluating it, through an example.

Some important points to note:
 - This guide is mainly written for [CS-773 (*Computer Architecture for Performance and Security*)](https://www.cse.iitb.ac.in/~biswa/courses/CS773/main.html) 
course at IIT Bombay, taught by [Prof. Biswabandan Panda](https://www.cse.iitb.ac.in/~biswa/) and I'm one of
the TAs of this course. Due to heterogeneity of the students taking this course (although majority are from CSE and EE departments) and varying experiences,
this guide assumes no prior knowledge, apart from knowing to use the shell, basic C++ and some fundamentals of computer-architecture and hence is **extremely verbose**.
Feel free to skip through the parts if you're already familiar with the ropes.
 - I'm not an *expert* since, at the time of writing this guide, I'm a 2nd year M.Tech. CSE student here. So there might be some flaws in my understanding
 here and there. If you find any issues, feel free to raise an issue and I'll try to fix it 🙂
 - This guide doesn't cover every aspect of ChampSim. If you have any such queries or other issues, please contact the devs of ChampSim.
 
**Update (2nd Feb, 2022)**: If you look at ChampSim's repository, you'll find that the build process, the prefetching API etc.
are different from what's written in this guide. ChampSim went through a big overhaul and more than a year worth of updates
have been (finally) merged into main branch, around a week back. This guide will be updated accordingly in the future, but for now,
use [this version of ChampSim](https://github.com/cs-773/ChampSim).    
 
## Table of Contents
- [Preliminaries - Getting Started](0_preliminaries/)
- [ChampSim's Prefetching API]()
- [Let's Write a Prefetcher! - Problem Statement]()
- [Let's Write a Prefetcher! - Implementation]()
- [Evaluating a Prefetcher - How Did it Perform ?]()
- [Visualizing the Results - An Image is Worth Thousand Numbers !]()
- [Where to Go Next - Machine Learning for Prefetching !?]()