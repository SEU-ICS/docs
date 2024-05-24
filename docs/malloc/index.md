# Malloc Lab

> Instructor: *Prof. Zhen Ling*

## Introduction

In this lab, you will be writing a dynamic storage allocator for C programs -- that is, your own version of the `malloc`, `free`, `realloc`, and `calloc` functions. You are encouraged to explore the design space creatively and implement an allocator that is *correct*, *efficient*, and *fast*.

***You are recommended to start by implementing an implicit list allocator. Subsequently, you can build on this foundation to implement other allocators (e.g., explicit list) to improve the performance of your heap allocator.***

## Environment Setup

The code required for the malloc lab in hosted on the same GitHub repository in the *shell lab*. The following are the basic steps to setup your local experiment environment:

* **Get the code**:
    * *downloading through the web page*: Please refer to the [shell lab guidance](../shell/index.md).
    * *using `git`*: Execute `git pull` under the code directory if you downloaded the code using `git clone` in the previous *shell lab*. Otherwise, execute `git clone https://github.com/SEU-ICS/lab.git`.
* **Enter the lab directory**: change your working directory to `lab/malloc` (or `lab-main/malloc`)

## Task Description
You will need to implement a dynamic storage allocator within `mm.c` by implementing the functions listed in Table-1. You may need to define other static helper functions used by the listed routines.

<center>
<b>Table-1. Malloc Lab Functions to be Implemented</b>
</center>

| Function Names | Descriptions |
| --- | --- |
|`int mm_init(void)` | The exposed API that performs any necessary initializations, such as allocating the initial heap area. The return value should be -1 if there was a problem in performing the initialization, 0 otherwise.<br> Note taht every time the driver executes a new trace, it resets your heap to the empty heap by calling your mm init function. |
|`void *malloc(size_t size)` | The exposed API that allocates memory chuncks. This routine returns a pointer to an allocated block payload of at least `size` bytes. The entire allocated block should lie within the heap region and should not overlap with any other allocated chunk. |
|`void free(void *ptr)` | The exposed API that frees allocated memory chunks. This routine frees the block pointed to by `ptr`. It returns nothing. This routine is only guaranteed to work when the passed pointer (i.e., `ptr`) was returned by an earlier call to `malloc`, `calloc`, or `realloc` and has not yet been freed. `free(NULL)` has no effect. |
|`void *find_fit(size_t asize)` | An internal function that locates a free memory chunk, used by `malloc()`. Find a block through all free blocks that meet the requirement of `asize`. |
|`void place(void *bp, size_t asize)` | An internal function that places the requested block in the new free block., used by `malloc()` |
| `void *extend_heap(size_t words)` | An internal function that extends the heap by `words` words. It returns a pointer to the new free block on success, `NULL` otherwise.  |
| `void *coalesce(void *bp)` | An internal function that merges two adjacent free memory chunks and returns the merged block. We have handled the simple case where both the previous and the next chunks are allocated. You will need to implement other cases. |

We have already implemented some helper routines in `memlib.c` which you can use:

* `void *mem_sbrk(int incr)`: Expands the heap by `incr` bytes, where `incr` is a positive non-zero integer, and returns a generic pointer to the first byte of the newly allocated heap area. The semantics are identical to the Unix `sbrk` function, except that `mem_sbrk `accepts only a positive non-zero integer argument.
* `void *mem_heap_lo(void)`: Returns a generic pointer to the first byte in the heap.
* `void *mem_heap_hi(void)`: Returns a generic pointer to the last byte in the heap.
* `size t mem_heapsize(void)`: Returns the current size of the heap in bytes.
* `size t mem_pagesize(void)`: Returns the system’s page size in bytes (4K on Linux systems).

We have also provided the basic implementations of `calloc` and `realloc` in `mm.c`. *However, you may need to modify these routines to improve the performance of your implementation.*

## Checking Your Work

### The Trace-driven Driver Program

The driver program `mdriver` (compiled from `mdriver.c`) tests your `mm.c` implementation for correctness, space utilization, and throughput. The driver program is controlled by a set of *trace files* that are included in the `traces` folder. Each trace file contains a sequence of allocate and free directions that instruct the driver to call your `malloc` and `free` routines in some sequence. The driver and the trace files are the same ones we will use when we grade your handin `mm.c` file.

When the driver program is run, it will run each trace file *12* times: once to make sure your implementation is correct, once to determine the space utilization, and *10* times to determine the performance.

If you run `mdriver` with no command line arguments, it will test the correctness and performance of your `mm.c` implementation by default. It also accepts the following command line arguments, which you may find useful during the development.

- `-t <tracedir>`: Look for the default trace files in directory tracedir instead of the default directory defined in `config.h`.

- `-f <tracefile>`: Use one particular tracefile instead of the default set of tracefiles for testing correctness and performance.

- `-c <tracefile>`: Run a particular tracefile exactly once, testing only for correctness. This option is extremently useful if you want to print out debugging messages.

- `-h`: Print a summary of the command line arguments.

- `-l`: Run and measure libc malloc in addition to the student’s malloc package. This is interesting mainly to see how slow the libc malloc package is.

- `-V`: Verbose output. Print additional diagnostic information as each trace file is processed. Useful during debugging for determining which trace file is causing your malloc package to fail.

- `-v <verbose level>`: This optional feature lets you manually set your verbose level to a particular integer.

- `-d <i>`: At debug level `0`, very little validity checking is done. This is useful if you are mostly done but just tweaking performance. At debug level `1`, every array the driver allocates is filled with random bits. When the array is freed or reallocated, we check to make sure the bits have not been changed. This is the default. At debug level `2`, every time any operation is done, all arrays are checked. This is very slow but useful to discover problems very quickly.

- `-D`: Equivalent to `-d2`.

- `-s`: Time out after s seconds. The default is to never time out.

- `-P`: Only output perf score to `stdout`.

### Correctness Check
To check the correctness of your heap implementation on all trace files, execute:
```shell
./scripts/exhaustive-heap-check.sh
```

To check the correctness of your heap implementation on only one trace files, execute:
```shell
./mdriver -c ./traces/<name>.rep
# e.g., ./mdriver -c ./traces/binary.rep
```

### Performance Check
To check the performance of your heap implementation, execute:
```shell
./scripts/get-perf-index.sh
```

## Programming Rules

- You should not change any of the interfaces in `mm.h`. However, we strongly encourage you to use static helper functions in `mm.c` to break up your code into small, easy-to-understand segments.
- You should not invoke any external memory-management related library calls or system calls. The use of the libc `malloc`, `calloc`, `free`, `realloc`, `sbrk`, `brk`, or any other memory management packages is strictly prohibited.
- You are not allowed to define any global data structures such as arrays, structs, trees, or lists in your `mm.c` program. However, you *are* allowed to declare global scalar variables such as integers, floats, and pointers in `mm.c`.

## Grading
The grading of the final hand-in will be based on the *correctness* and *performance* of your allocator on the given traces, the quality of your heap checker, and your *coding style*:

* **Correctness (80 points)**: we use *20* trace files as test cases for correctness, each test case counts for *4* points, and you will get a total of *80* points if you pass all tests. Use the correctness check script mentioned above to estimate your correctness score.
* **Performance (100 points)**: Two metrics will be used to evaluate your solution. Use the performance check script to estimate your performance score.
    * *Space utilization*: The peak ratio between the aggregate amount of memory used by the driver (i.e., allocated via `malloc` but not yet freed via `free`) and the size of the heap used by your allocator. The optimal ratio equals 1. You should find good policies to minimize fragmentation in order to make this ratio as close as possible to the optimal.
    * *Throughput*: The average number of operations completed per second.

Your final grade is calculated by the following formula. The valuation of the weighting parameters $a$ and $b$ depends on the overall performance of the class.

$$Grade = a \times CorrectnessScore + b \times PerformanceScore$$

## Hand-In Instructions
We use GitHub Classroom to manage and organize labs. Follow these steps to submit your `mm.c`:

* **Join the GitHub Classroom**: You can safely jump to the next step if you have joined the classroom. Otherwise, an invitation link has been shared in the course group chat, open the link to join the GitHub Classroom. You'll need to register a GitHub account if you don't have one. You should be able to see your student ID (e.g., `09Jxxxxx`) listed in the roaster, please carefully find your student ID and link your GitHub account with it.
* **Accept the assignment**: An invitation link has been shared in the course group chat, you will need to open the link to accept the assignment. A private repository will be created for you once you accept the assignment.
* **Submit your work**: Submit your `mm.c` file to your repository with a commit. You can submit as many times as you want before the deadline. Please refer to the [shell lab guidance](../shell/index.md) for how to submit.
* **Grading**: Our auto-grading tool will automatically evaluate your submissions every time you push a commit. Only you, teachers and TAs can view the score of your submission. The score that auto-grader reports is the sum of your correctness score and performance score (i.e., $a=1, b=1$) and does not reflect your final grade. However, this score is guaranteed to positively correlates with your final grade -- higher score, higher final grade. We will use the score of your final submission to calculate your final grade for this lab.

**NOTE**:

1. You will not be able to submit your work after the deadline. Manage your time properly.
2. The online auto-grader runs slow and should only be used to obtain your final score. Use the provided scripts locally to debug and evaluate your `mm.c`.
3. Any modifications to the `.github` directory of your repository is **STRICTLY PROHIBITED**.
4. Code plagiarism is **STRICTLY PROHIBITED AND PUNISHED**.

Further instructions, if any, will be announced in form of class group notifications.