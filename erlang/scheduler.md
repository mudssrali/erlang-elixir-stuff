# Erlang Scheduler: what does it do?
```txt
On Sun, 22 Apr 2001, James Vernon wrote:

>Hi,
>
>Can someone tell me how Erlang processes are scheduled? I'm doing
>some quick research into real-time languages, I know Erlang is only
>a `soft' real-time system, but just how soft is it? Does Erlang make
>any attempt at all to meet real-time deadlines other than making
>context switches very fast and efficient?
```

Erlang processes are currently scheduled on a reduction count basis.
One reduction is roughly equivalent to a function call.
A process is allowed to run until it pauses to wait for input (a
message from some other process) or until it has executed 1000
reductions.

There are functions to slightly optimize the scheduling of a process
(yield(), bump_reductions(N)), but they are only meant for very
restricted use, and may be removed if the scheduler changes.

A process waiting for a message will be re-scheduled as soon as there
is something new in the message queue, or as soon as the receive timer
(receive ... after Time -> ... end) expires. It will then be put last
in the appropriate queue.

Erlang has 4 scheduler queues (priorities): 
`max`, `high`, `normal`, and `low`.


'max' and 'high' are strict. This means that the scheduler will
first look at the 'max' queue and run processes there until the
queue is empty; then it will do the same for the 'high' queue.

'normal' and 'low' are fair priorities. Assuming no processes at
levels 'max' and 'high', the scheduler will run 'normal' processes
until the queue is empty, or until it has executed a total of 8000
reductions (8*Max_Process_Reds); then it will execute one 'low'
priority process, if there is one ready to run.

The relationship between 'normal' and 'low' introduces a risk of
priority inversion. If you have hundreds of (active) 'normal'
processes, and only a few 'low', the 'low' processes will actually get
higher priority than the 'normal' processes.

There was an experimental version of a multi-pro Erlang runtime
system. It supported Erlang processes as OS threads, and then used the
scheduling and prority levels of the underlying OS.


One important thing to note about the scheduling is that from the
programmer's perspective, the following three things are invariant:

- scheduling is preemptive.
- a process is suspended if it enters a receive statement, and there
  is no matching message in the message queue.
- if a receive timer expires, the process is rescheduled.

The Erlang Processor, for example, will most likely schedule based on
CPU cycles (essentially a time-based scheduler). It may also have 
multiple thread pools, allowing it to context switch within one 
clock cycle to another process if the active process has to e.g.
wait for memory.

In a runtime environment that supports it, it should be possible
to also have erlang processes at interrupt priority (meaning that 
they will be allowed to run as soon as there is something for them
to do -- not having to wait until a 'normal' priority process finishes
its timeslice.)

```
/Uffe
-- 
Ulf Wiger                                    tfn: +46  8 719 81 95
Senior System Architect                      mob: +46 70 519 81 95
Strategic Product & System Management    ATM Multiservice Networks
Data Backbone & Optical Services Division      Ericsson Telecom AB
```