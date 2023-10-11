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

## Task 2/3: Priority Scheduling and Advanced Scheduler

The suggested order of implementation for this task is:

1. Basic support for priority scheduling in Exercise 2.1 when a new thread is created in `thread_create()`.
2. Implement fixed-point real arithmetic routines (B.6 Fixed-Point Real Arithmetic) that are needed for MLFQS in exercise 3.1
3. Add full support for priority scheduling in Exercise 2.1 by considering all other possible scenarios and the synchronization primitives.
4. Tackle either Exercise 3.1 first or Exercise 2.2 first.

### Thread Creation Support

(Reference to test: `alarm-priority`).

From the name of the test we may infer that this would likely relate to the priority problems in running thread.

By observing the failed output results for `alarm-priority` test, we can see that the problem is the Threads that get unblocked did not run in a priority-order.

We then go to the implementation of picking running threads: they automatically picks the front of the `ready_list`.

```c
/* Chooses and returns the next thread to be scheduled.

Should return a thread from the run queue, unless the run queue is empty.  

(If the running thread can continue running, then it will be in the run queue.)  
If the run queue is empty, return idle_thread. */
static struct thread *
next_thread_to_run (void) 
{
  if (list_empty (&ready_list))
    return idle_thread;
  else
    return list_entry (list_pop_front (&ready_list), struct thread, elem);
}
```

Threads are inserted into this list whenever `thread_unblock()` or `thread_yield()` are called. The prior change a previous thread in `BLOCKED` state into `READY` and add it into the list. The latter change a currently `RUNNING` thread into `READY` state as it yields the CPU to other thread users.

To achieve priority ordering, we would again use `list_insert_ordered()` to ensure each thread inserted into the list follows such order.

---

After that we came back to Question 2.1's description on thread yielding.

> Exercise 2.1
>
> Implement priority scheduling in Pintos.
>
> - When a thread is added to the ready list that has a higher priority than the currently running thread, the current thread should immediately yield the processor to the new thread.
> - Similarly, when threads are waiting for a lock, semaphore, or condition variable, the highest priority waiting thread should be awakened first.
> - A thread may raise or lower its own priority at any time, but lowering its priority such that it no longer has the highest priority must cause it to immediately yield the CPU.

The 3rd point would be the easiest to cope with: upon change of thread priority in `thread_set_priority`, we compare the new priority with its original priority. If the priority has decreased then we immediately call `thread_yield()` to yield the CPU.

By reading the comments we discover that `thread_unblock` shall not try to preempt the current thread as it may be called under interrupt disabled mode.

- call `intr_yield_on_return()` to yield the CPU under an interrupt context.

> Exercise 2.2.2
>
> Support nested priority donation:
>
> If `H` is waiting on a lock that `M` holds and `M` is waiting on a lock that `L` holds, then both `M` and `L` should be boosted to `H`'s priority.
>
> If necessary, you may impose a reasonable limit on depth of nested priority donation, such as 8 levels.
>
> **Note:** if you support nested priority donation, you need to pass the priority-donate-nest and priority-donate-chain tests.

### Priority Donation

In `struct thread`:

- [x] Support a differentiation between displayed priority (`int priority`) and its actual priority (`int base_priority`).
  - Displayed priority `int priority` represents the priority behaviour of the thread at the current moment. This may not come from the thread itself but from priority donations, nested donations, etc.
  - Its actual priority `int base_priority` is the TRUE priority occupied by the thread itself, regardless of any lock dependencies and priority donations.
- [x] Support corresponding `thread_set_priorty` changes. The set priority now will update its `base_priority`, however, whether the displayed `priority` should be updated needs to be discussed.
  - Scenarios to update `priority`: the thread is not holding any locks or the new given `new_priority` is higher than that of the current `base_priority`
- [ ] Support a list of locks held by the current thread `list my_locks`.
  - [x] Create object definition.
  - [ ] Once the thread acquired any lock, add this particular lock to `my_locks`, in the order of `thread_priority` (tips: write a helper function and use `list_insert_ordered`)
  - [ ] Once a lock is released, goes to the `my_locks` list and the corresponding thread holder, update that thread's display priority `int priority`.
  - [ ] Be careful when handling change of `priority` when the lock has been released: it does not directly go back to `base_priority` but instead pick the front of the queue in its `my_locks` list.
- [ ] Support a `lock` pointer `struct lock *current_lock` for current thread to show whether it is waiting for response from another `lock`, in case nested priority donation needs it.

In `synch.c`:

Consider priority donation: priority donation will only happen when a thread, namely `t0`, requests another lock, so it's in `lock_acquire`.

Before acquiring the lock:

- [ ] update `current_lock` of `t0` to this lock we are acquiring.
- [ ] If this lock is currently held by other thread, and the current holder `t1` has a lower priority compared to `t0`: Then priority donation happens. Meanwhile the `max_priority` of the lock should also be updated.

After acquiring the lock:

- [ ] `list_insert_ordered` this lock into `my_locks` of `t0`.

Similarly, when we are releasing a lock:

Before manipulating the semaphore:

- [ ] Resetting `t0`'s priority.
