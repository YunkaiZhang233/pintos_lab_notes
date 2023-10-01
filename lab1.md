# Lab 1: Threads

## References

This set of lab notes is based on the [UCB CS-162 Fall 2020](https://inst.eecs.berkeley.edu/~cs162/fa20/) class,  [JHU CS-318 Project 1: Getting Real](https://www.cs.jhu.edu/~huang/cs318/fall21/project/project1.html) and the [Pintos Lab Guide](https://alfredthiel.gitbook.io/pintosbook/) provided by Peking University.

See the website for background description.

From now on the detailed description of the tasks will be skipped as they are already clearly described in the websites.

## Task 1: Alarm Clock

### Exercise 1.1

Analyze the most crucial piece of code for busy-waiting version:

```c
/** Sleeps for approximately TICKS timer ticks.  Interrupts must
   be turned on. */
void
timer_sleep (int64_t ticks) 
{
  int64_t start = timer_ticks ();

  ASSERT (intr_get_level () == INTR_ON);
  while (timer_elapsed (start) < ticks) 
    thread_yield ();
}
```

As other variations of sleeping will be eventually redirected into `timer_sleep`.

For this function, `time_ticks()` fetches number of ticks since booting and `timer_elapsed()` subtracts the current `time_ticks()` from the passed-in parameter `ticks` to get the numerical time elapsed.

We can clearly see that the original method _busy waits_ as it uses a tight loop calling `thread_yield` to continuously `yield` the current thread and then try until the required time has elapsed. Calling `thread_yield` essentially switches operating thread in the CPU but turns the current thread itself into `READY` state.

To get rid of busy waiting, we will aim to stop 'yielding' itself and instead it shall switch its state to `BLOCKED` and wait until some higher-level program that manages time changing and will wake it when the required time had elapsed.

Observing the program we found out that in the initialisation process,

```c
/** Sets up the timer to interrupt TIMER_FREQ times per second,
   and registers the corresponding interrupt. */
void
timer_init (void) 
{
  pit_configure_channel (0, 2, TIMER_FREQ);
  intr_register_ext (0x20, timer_interrupt, "8254 Timer");
}
```

Following through, we found that `timer_interrupt` function actually acts as the handler for time interrupts, and the CPU calls this function in every time segment for timing updates.

Finding this structure, we can choose to perform the following operations:

- Record the information of threads and their required sleeping time in `timer_sleep`
- Switch to `BLOCKED` state for the current thread. Add thread info to a list
- In each call to `timer_interrupt`, update the remaining time info for all threads in the list.
- Loop through the list and put the first thread that had reached its required time into `UNBLOCKED` status and execute accordingly.
