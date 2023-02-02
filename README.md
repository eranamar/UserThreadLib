# UserThreadLib

This is my Exercise NO.2 written during OS course at HUJI spring 2013.
In this exercise I implemented a user-level thread library with a basic 
Round-Robin (RR) scheduling algorithm. The package is delivered in the 
form of a static library. The public interface of the library is `uthreads.h`.

## The Library 's Description

<article>

<section>

#### <a href="">Assignment</a>

This is my Exercise NO.2 written during <abbr title="Operation System">OS</abbr> course at <abbr title="Hebrew University of Jerusalem">HUJI</abbr> spring 2013.  
In this exercise I implemented a user-level thread library with a basic Round-Robin (RR) scheduling algorithm. The package is delivered in the form of a static library. The public interface of the library is `uthreads.h`.

</section>

<section>

#### <a href="">The Threads</a>

Initially, a program comprises of the default main thread, whose ID is 0\. All other threads will be explicitly created. Each existing thread has a unique thread ID, which is a non-negative integer. The ID given to a new thread is the smallest non-negative integer not already taken by an existing thread. The maximal number of threads the library supports (including the main thread) is `MAX_THREAD_NUM`.

</section>

<section>

#### <a href="">Thread State Diagram</a>

<div>  
At any given time during the running of the user's program, each of the threads in the program is in one of the states shown in the following state diagram. Transitions from state to state occur as a result of calling one of the library functions, or from elapsing of time, as explained below.  
![State diagram picture](imgs/uthread_stateDiagram.png)</div>

</section>

<section>

#### <a href="">The Scheduler</a>

<div>The scheduling algorithm I uses in this exercise is a simple Round Robin (RR). First, note that whenever I mention "time" in this file, I mean the running time of the process (virtual time), and not the real time that has passed in the system. The process running time is measured by the Virtual Timer.  
The round robin scheduling policy is as follows:

*   Every time a thread is moved to the RUNNING state, it is allocated a predefined number of micro-seconds to run. This time interval is called a quantum.
*   A thread is preempted if any of the following occurs:
    *   Its quantum expires
    *   It is suspended
    *   It is put to sleep
    *   It is terminated
*   If the RUNNING thread is preempted for any reason, the next thread in the list of READY threads is moved to the RUNNING state.
*   Every time a thread moves to the READY state from any other state, it is placed at the end of the list of READY threads.
*   There are no 'compensations' for quantums that were not fully used (as in the case of a thread that suspended itself).

</div>

</section>
</article>

### Library API

<article><a href="" class="Fold_all">Fold</a> <a href="" class="expand_all">Expand</a>  

Following is the list and descriptions of all library functions. Calling these functions may result in a transition of states to one of the threads. A thread may call a library function with its own ID, thereby possibly changing its own state, or it may call a library function with some other thread's ID, thereby affecting the other thread's state.  

<section>

#### <a href="">`int uthread_init(int quantum_usecs)`</a>

**Description:** This function initializes the thread library. I am assuming that this function is called before any other thread library function, and that it is called exactly once. The input to the function is the length of a quantum in micro-seconds.  
**Return value:** On success, return 0\. On failure, return -1.

</section>

<section>

#### <a href="">`int uthread_spawn(void (*f)(void))`</a>

**Description:** This function creates a new thread, whose entry point is the function `f` with the signature `void f(void)`. The `uthread_spawn` function should fail if it would cause the number of concurrent threads to exceed the limit (`MAX_THREAD_NUM`). Each thread is being allocated with a stack of size `STACK_SIZE` bytes.  
**Return value:** On success, return the ID of the created thread. On failure, return -1.

</section>

<section>

#### <a href="">`int uthread_terminate(int tid)`</a>

**Description:** This function terminates the thread with ID `tid` and deletes it from all relevant control structures. All the resources allocated by the library for this thread will be released. Terminating the main thread (tid == 0) will result in the termination of the entire process using `exit(0)`.  
**Return value:** The function returns 0 if the thread was successfully terminated and -1 otherwise. If a thread terminates itself or the main thread is terminated, the function does not return.

</section>

<section>

#### <a href="">`int uthread_suspend(int tid)`</a>

**Description:** This function suspends the thread with ID `tid`. The thread may be resumed later using `uthread_resume`. It is an error to try to suspend the main thread (tid == 0). If a thread suspends itself, a scheduling decision will be made. Suspending a thread in the SUSPENDED or SLEEPING state has no effect and is not considered an error.  
**Return value:** On success, return 0\. On failure, return -1.

</section>

<section>

#### <a href="">`int uthread_resume(int tid)`</a>

**Description:** This function resumes a suspended thread with ID `tid` and moves it to the READY state. Resuming a thread in the RUNNING, READY or SLEEPING state has no effect and is not considered an error.  
**Return value:** On success, return 0\. On failure, return -1.

</section>

<section>

#### <a href="">`int uthread_sleep(int num_quantums)`</a>

**Description:** This function puts the RUNNING thread to sleep for a period of `num_quantums` after which it is moved to the READY state. `num_quantums` must be a positive number. It is an error to try to put the main thread to sleep. Immediately after a thread transitions to the SLEEPING state a scheduling decision should be made.  
**Return value:** On success, return 0\. On failure, return -1.

</section>

<section>

#### <a href="">`int uthread_get_tid()`</a>

**Description:** This function returns the thread ID of the calling thread.  
**Return value:** The ID of the calling thread.

</section>

<section>

#### <a href="">`int uthread_get_total_quantums()`</a>

**Description:** This function returns the total number of quantums that were started since the library was initialized, including the current quantum. Right after the call to `uthread_init`, the value should be 1\. Each time a new quantum starts, regardless of the reason, this number should be increased by 1.  
**Return value:** The total number of quantums.

</section>

<section>

#### <a href="">`int uthread_get_quantums(int tid)`</a>

**Description:** This function returns the number of quantums that were started for the thread with ID `tid`, including the current quantum. On the first time a thread runs, the function should return 1\. Every additional quantum that the thread starts should increase this value by 1.  
**Return value:** On success, return the number of quantums of the thread with ID `tid`. On failure, return -1.

</section>
</article>

### Simplifying Assumptions

<article>

1.  All threads end with uthread_terminate before returning, either by terminating themselves or due to a call by some other thread.
2.  The stack space of each spawned thread isn't exceeded during its execution.
3.  The main thread and the threads spawned using the uthreads library will not send timer signals themselves (specifically `SIGVTALRM`) or set interval timers that do so.

</article>

### Error Messages

<article>

When a system call fails I prints a single line to stderr in the following format:"system error: `text`\n" Where `text` is a description of the error, and then `exit(1)`.  
When a function in the threads library fails, I prints a single line to stderr in the following format: "thread library error: `text`\n" Where `text` is a description of the error, and then return the appropriate return value.

</article>

### Makefile

<article>

The makefile generates a static library file named: `libuthreads.a` when running `make` with no arguments.




