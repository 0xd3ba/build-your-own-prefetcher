# Let's Write a Prefetcher! - Implementation

Assuming you have already tried building your prefetcher based on the problem statement described in
"[Let's Write a Prefetcher! - Problem Statement](../2_write_a_prefetcher_p1/)", this part will go through the solution
to the problem. If you couldn't come up with an implementation previously, then that's fine - maybe my problem statement
was not clear enough :)

### Part-1: Analyzing the Requirements
It's always good to start thinking about the APIs of whatever modules your prefetcher is gonna use, before jumping to code directly. Just like any other project,
the difficulty to refactor and/or debug increases as the code size increases. Although it might not apply to this specific example, 
it's always better to maintain a good habit. Anyways, going through the problem statement, following  requirements can be inferred
for the prefetcher

- Some appropriate data-structure to represent the history buffer
- A method to indicate whether the history buffer is full or not
- A method to append new items to the buffer, irrespective of the internal data-structure used
- A method to reset the buffer, irrespective of the internal data-structure used
- A method to check if the buffer is full or not
- A method to check if there is a forward-movement or not, in the buffer

And we are going to need some utility methods/macros to
- Represent the buffer size `K`
- Extract the block ID and page ID from the address in `l1d_prefetcher_operate` method
- Check if the current access is in the same page as the previous access
- Prepare the address to prefetch, based on the predicted block `(X+1)` or `(X-1)`

All of these should be put in a separate header-file, say, `next_line_remix.h` inside ChampSim's `inc/` directory.
Although it's much better to split across mutiple source/header files, ChampSim expects the prefetcher to be defined
in a **single** `*.l*_pref` file. Although there is no such restriction on the header files, to reduce clutter, we'll use a single header file.
(PS: There are no such restrictions in the newer version of ChampSim and code can be organized in a clutter-free manner)

### Part-2: Writing the Utility Methods and Macros
Writing the utility methods and macros are pretty simple. We'll also be using `champsim.h` header file to use few pre-defined
macros related to page size and block size to make the code more general-purpose and robust. Define the following in a separate
header file, as mentioned previously.

```C++
#include "champsim.h"

/* Convenient macros to extract out page ID and block ID from a given 64-bit address */
#define EXTRACT_PAGE_ID(addr)   ((addr) >> LOG2_PAGE_SIZE)              /* Extract the page ID */
#define EXTRACT_BLOCK_ID(addr)  (((addr) >> LOG2_BLOCK_SIZE) & 0x3f)    /* Extract the block ID within the page */
 
/* Minimum and maximum value of the block IDs */
#define BLOCK_ID_MIN    0
#define BLOCK_ID_MAX    ((PAGE_SIZE / BLOCK_SIZE) - 1)

/* The length of the buffer, i.e. 'K' as mentioned in the question */
/* Tweak this value accordingly */
#define BUFFER_LENGTH   5

/* Utility method to prepare the address to prefetch */
uint64_t prepare_prefetch_address(uint64_t page_id, uint32_t block_id) {
    return (page_id << LOG2_PAGE_SIZE) + (block_id << LOG2_BLOCK_SIZE);
}
``` 

Checking if the previous access was to the same page requires us to track the last-accessed page in a separate variable.
A thing to note is that ChampSim also supports multi-core simulations. Each core has its own private L1-D cache and so are the prefetchers.
To disambiguate between the prefetchers (and its corresponding values), we need to define multiple copies, wherever necessary.
Although this is not required for single-core simulations, writing general-purpose code is much better.
This might not make sense at the moment, but will be so, later on below.

```C++
/* NUM_CPUS is a pre-defined constant set to the number of cores to use in the simulation */
uint64_t prev_page_id[NUM_CPUS];    /* Store the IDs of the previous page, per prefetcher */

/* Returns True if the access is to the same page. False otherwise */
bool is_on_same_page(uint32_t cpu_id, uint64_t curr_page_id) {
    uint64_t tmp_prev_page_id = prev_page_id[cpu_id];   /* Temporarily store the value of previous page ID */
    prev_page_id[cpu_id] = curr_page_id;                /* Update the page ID of the latest page accessed */
    
    /* If previous page ID is same as the current page ID */
    if(curr_page_id == tmp_prev_page_id)
        return True;
    
    /* Nope, it's different. Return False */
    return False;
}
```

### Part-3: Writing the History Buffer
For this example, this is the only module that the prefetcher will be using and as described in part-1, we already
have an idea on what methods we need to define (what about the data-structure for the buffer ?).

```C++
class HistoryBuffer {
    /* Add in your variables for the buffer */
    /* Hint: You might wanna see deque data-structure */

public:

    HistoryBuffer() {
        /* Write your code */
    }


    /* size -- Returns the current size of the buffer */
    int32_t size() {
        /* Write your code */
    }

    /* append -- Method that takes a cache block # as input and pushes it to the back of the buffer */
    void append(uint32_t blk_id)
    {   
        /* Ensure that the buffer stores only K most-recent entries */
        /* Write your code */
    }


    /* reset -- Method that resets the corresponding buffer by removing all the elements */
    void reset()
    {
        /* Write your code */
    }


    /* is_full -- Returns TRUE if the buffer is full. FALSE otherwise */
    bool is_full() {
        /* Write your code */
    }


    /* has_forward_movement -- Method that returns TRUE if atleast ((K-1)/2 + 1) elements in the buffer
     *                         has a forward movement. Returns FALSE otherwise.
     *
     * NOTE: Ensure that this method is called only when the buffer is completely full to ensure
     *       correct results
     */
    bool has_forward_movement()
    {
        /* Write your code */
    }
};

/* Declare buffers per prefetcher */
HistoryBuffer hist_buff[NUM_CPUS];    /* History-Buffer Module for the prefetcher */ 
```

### Part-4: Writing the Prefetcher

Now that everything is set-up, we can finally fill in the ChampSim's prefetcher methods. Copy a template
and rename it accordingly (as described in previous part of the guide). For this example, the only methods that
we need to concern ourselves with are
- `l1d_prefetcher_initialize`
- `l1d_prefetcher_operate`

The remaining methods can be left empty, since we have no use for them. But they must be present in the `*.l1d_pref` file
(they have been omitted in the following snippet below).

```C++
#include "champsim.h"

/* Initialize the prefetcher's utility variables */
void CACHE::l1d_prefetcher_initialize() {
    for(int i=0; i<NUM_CPUS; i++)
        prev_page_id[i] = -1;    /* Set it to the maximum 64-bit value */
}


void CACHE::l1d_prefetcher_operate(uint64_t addr, uint64_t ip, uint8_t cache_hit, uint8_t type) {
    uint64_t page_id = EXTRACT_PAGE_ID(addr);        /* Extract out the page ID of the current load/store */
    uint32_t block_id = EXTRACT_BLOCK_ID(addr);      /* Extract out the block ID of the current load/store */
    int32_t prefetch_block_id = (int32_t) block_id;  /* Temporarily store the block ID that we'll increment/decrement later on */
    uint64_t prefetch_addr;                          /* Where we'll store the prefetch address later on */
    
    /* Check if the access is on the same page as previous page */
    /* If no, then reset the buffer and return */
    /* NOTE: cpu is a member variable of CACHE, which stores the ID of the CPU where the cache belongs */
    if(!is_on_same_page(cpu, page_id)) {
        hist_buff[cpu].reset();
        return;
    }
    
    /* Previous access was to the same page - We can continue */
    hist_buff[cpu].append(blk_id);                   /* Append the latest block ID to the buffer */
    
    /* If the history buffer is not full, we can't send a prefetch request */
    if(!hist_buff[cpu].is_full())
        return;
    
    /* History buffer is full -- We can continue */
    /* If the buffer has forward-movement, we will prefetch the next block. Else the previous block */
    if(hist_buff[cpu].has_forward_movement())
        prefetch_block_id++;
    else
        prefetch_block_id--;
    
    /* Now we can send a prefetch request, if the prefetch block falls in the same page */
    if(prefetch_block_id >= BLOCK_ID_MIN && prefetch_block_id <= BLOCK_ID_MAX) {
        prefetch_addr = prepare_prefetch_address(page_id, prefetch_block_id);   /* Prepare the address to prefetch */
        
        /* Finally send the prefetch ! */
        /* The prefetch will be filled till L1-D */
        /* Because we don't have any use for metadata, set it to 0 */
        prefetch_line(ip, addr, prefetch_addr, FILL_L1, 0);
    }

}
```

## Conclusion
By now, you should have got the feel for writing a prefetcher. Although the example was simple and straight-forward,
the state-of-the-art prefetchers are not as straight-forward and it's not unsurprising to see them use many
modules (similar to the `HistoryBuffer` module you saw previously, but more sophisticated). Designing a prefetcher for 
L2C and LLC are similar as well, so this can be easily ported to work with them as well. If you understood everything till
now, you'll have no issue in materializing your own prefetcher ideas into ChampSim - which was the main objective of this guide.
Although you have written the prefetcher, evaluation of it is still remaining. The next parts of this guide will describe it in detail.

## Exercises
1. Fill in the methods for `HistoryBuffer` class as shown previously. Think about an appropriate data-structure to use
for the buffer (Hint: `deque` is a good one).
2. Port this implementation to work with L2C and LLC. 