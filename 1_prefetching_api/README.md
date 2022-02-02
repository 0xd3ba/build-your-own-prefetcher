# ChampSim's Prefetching API

This part of the guide will assume that you are comfortable with building ChampSim and running simulations with it.
If not, then go through "[Preliminaries - Getting Started](../0_preliminaries/)" first. As discussed previously, all the
prefetchers in ChampSim are present in `prefetcher/` directory as C++ source files where

| Extension | Type |
|:------:|:-----|
|`*.l1i_pref` | L1-I Cache Prefetcher |
|`*.l1d_pref` | L1-D Cache Prefetcher |
|`*.l2c_pref` | L2 Cache Prefetcher |
|`*.llc_pref` | L3 Cache (or LLC) Prefetcher |
    
At a high-level view, the way to write a prefetcher is to
1. Copy an appropriate template (`no.l*_pref`), depending on the level on which you want your prefetcher to operate on
   and rename it to your prefetcher (while keeping the same extension)
2. Fill in the class methods appropriately (described in detail below) with your prefetcher's algorithm. 
   ChampSim will call those methods whenever necessary, during the simulation process.

The API for L1-D, L2C and LLC prefetchers are almost the same except for **L1-I**, which is not discussed at the moment
(left as a future work). The following sections describe each of those methods in detail.

### `l*_prefetcher_initialize()`
This method is self-explanatory from its name. It is invoked during initialization phase and it should be used to print
any information about the prefetcher (configuration for ex.) and/or initialize anything that the prefetcher 
cannot initialize statically during compile-time.

### `l*_prefetcher_operate(...)`
| Parameter | Brief Description |
|----------:|:------------|
| `addr`    | Physical address with which the corresponding cache was accessed |
| `ip`      | Instruction-pointer that was responsible for initiating the request |
| `cache_hit` | A flag indicating whether the above mentioned address resulted in a hit in the cache |
| `type`    | Whether the request is a demand-request or a prefetch request from a higher cache level |
| `metadata_in` | Any 64-bit information passed from the *higher* cache level's prefetcher (Not present in `l1d_prefetcher_operate`). Default value is 0 |

**Returns**:
- Nothing, in `l1d_prefetcher_operate`
- Any 64-bit (`uint64_t`) information that the prefetcher at L2C/LLC would like to pass on to lower level

This is **the** method that defines how your prefetcher operates, i.e. your prefetching algorithm must be defined in this
method. This method is called on every cache access, i.e. the prefetcher is invoked on every cache access and the prefetcher
can decide to send (or not send) any number of prefetches, as long as the prefetch queue is not full (otherwise the prefetch 
request(s) would be dropped).    

### `l*_prefetcher_cache_fill(...)`
| Parameter | Brief Description |
|----------:|:------------|
| `addr`    | Bits[6:63] of the address (to which a past prefetch request was issued) which has now been brought to the cache |
| `set`     | The set-number in the cache, to which this address maps to |
| `way`     | The way-number in the above set, to which this address maps to |
| `prefetch` | A flag indicating whether the address that will be evicted, was a prefetched address |
| `evicted_addr` | Bits[6:63] of the address (located at above set-way pair) which will be evicted |
| `metadata_in` | Any 64-bit information passed from the *lower* cache level's prefetcher. Default value is 0 |

**Returns**:
- Nothing, in `l1d_prefetcher_cache_fill`
- Any 64-bit (`uint64_t`) information that the prefetcher at L2C/LLC would like to pass on to higher level

This is the method which is called when a prefetch request gets resolved, i.e. the address to which a prefetch request
was sent in the past has now been brought to the cache and will be placed in the above mentioned `set` and `way`. Note that
this method is called **before** the replacement happens, i.e. the location at `set` and `way` still holds information about the previously
mapped address.

### `l*_prefetcher_final_stats()`
This method is self-explanatory as well. When the simulation ends, this method is invoked. It should be used to print any
statistics that were being tracked by the prefetcher and/or de-allocate any resources that were allocated during the
call to `l*_prefetcher_initialize()` method. 

---

Although the above methods are required to define your prefetcher, there is still one method, which although doesn't need to be
defined, but needs to be used to **send prefetch requests**. Assuming your prefetcher "magically" came up with an address
to prefetch during the call to `l*_prefetcher_operate` method, it still needs to send the prefetch request right ? The method below does the job

### `prefetch_line(...)`
| Argument | Brief Description |
|----------:|:-----------------|
| `ip`      | Must be the same value as the `ip` parameter in `l*_prefetcher_operate` method  |
| `base_addr` | Must be the same as the parameter `addr` in `l*_prefetcher_operate` method. This is basically used to validate if the prefetch falls outside of the page boundary (request is dropped if yes) |
| `pf_addr` | The **cache-aligned** address to which the prefetch request is being issued. Bits[0:5] must be 0 (since block size is 64B) |
| `pf_fill_level` | The cache-level till which the prefetched address will be filled. Must be set to either of `FILL_L1`/`FILL_L2`/`FILL_LLC`
| `metadata_in`   | Any 64-bit information that needs to be passed to the *lower* cache level's prefetcher. Default is 0 | 

**Returns**:
- `1` if prefetch request is successfully sent
- `0` if the prefetch request is dropped, due to any of the reasons mentioned previously


## Conclusion
In the above sections, ChampSim's prefetching API was discussed briefly. If most of the things above didn't make sense, it's
completely fine since we'll go through an end-to-end prefetcher implementation step-by-step in the next two parts of the guide.
Refer to this part of this guide whenever something doesn't make sense and/or you forget parts of the API. 

Have fun :)