# Let's Write a Prefetcher! - Problem Statement

From this part of the guide, we'll finally get our hands dirty with prefetcher implementation, simulate it on few benchmarks 
and then evaluate its performance against a system with no prefetching.
Make sure that you have gone through the previous parts of the guide before moving on ahead. 

Alright, let's begin!

## Problem Statement
One fine winter evening, Golu, an aspiring computer-architecture researcher, was skimming through various kinds of data 
prefetchers available for CPU caches because he had the idea out of nowhere to try building one of his own, after attending 
a CS-773 lecture during the daytime. Because he was completely new to the field of prefetching, he quickly got overwhelmed 
by most of the sophisticated approaches and hence decided to improve upon a simple prefetcher: the *Next-Line Prefetcher*.

Golu liked the simplistic nature of the next-line prefetcher:
> When a demand request for cache-line # `X` arrives in the cache, 
> prefetch cache-line # `(X+1)`, as long as it falls inside the same page

and hence had the idea to make it behave more *smartly*, atleast according to him. He thought, maybe instead of prefetching 
`(X+1)` all the time, why not also support prefetching `(X-1)`, depending on the *direction* of the demand requests ?
More specifically, he had the idea that, if for example, the demand request history of most-recent `K (>= 2)` cache-line #s 
(within the **same** page), in sequential order, are 

<table>
  <tr>
    <th> Cache-Line # </th>
    <td> 10 </td>
    <td> 15 </td>
    <td> 14 </td>
    <td> ... </td>
    <td> 20 </td>
    <td> 23 </td>
  </tr>
  
  <tr>
    <th> Time-Step </th>
    <td> (T-K+1) </td>
    <td> (T-K+2) </td>
    <td> (T-K+3) </td>
    <td> ... </td>
    <td> (T-1) </td>
    <td> T </td>
  </tr>
</table>

then his proposed prefetcher would prefetch cache-line # 24 if there are at least `(K-1)/2 + 1` *"forward-movements"*, 
else would prefetch cache-line # 22. In his terms,

> A forward-movement is when the difference between cache-line # at time-step `t` and cache-line # at `(t-1)` is 
> positive, i.e. `>= 0`

In the above example, `10 -> 15`  is a forward-movement because the difference is `+5`. However  `15 -> 14` is not a 
forward-movement because the difference is `-1`. Because he did not want to complicate things, he decided that 

- If there is a change in the page (i.e. the previously accessed cache-line is in a different page compared to current cache-line), 
his prefetcher would **reset** the observed history and would start accumulating the history again from scratch and hence, his prefetcher 
will only start sending prefetches again only when the accumulated history becomes `>= K` cache-lines again
- If the cache-line # to prefetch falls outside page-boundary, then his prefetcher **will not** issue a prefetch request
- His prefetcher would operate at **L1-D** cache

Can you help Golu design the prefetcher (for `K = 5`) and help him evaluate its performance when compared against a system 
with no prefetcher and the traditional *Next-Line Prefetcher* ?

## Exercises
1. Try building the prefetcher on your own without referring to the solution (described in the next part). Feel free to 
go though the previous two parts of the guide, where basics of ChampSim and the prefetching API were described.