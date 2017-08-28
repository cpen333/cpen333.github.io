---
layout: lecture
title:  Introduction to Concurrent Systems
date:   2017-09-07 17:50:00
authors: [Paul Davies, C. Antonio Sánchez]
categories: [lectures, systems, concurrency]
usemath: true

---

# Introduction to Concurrent Systems
{:.no_toc}

**Authors:** Paul Davies (UBC), C. Antonio Sánchez (UBC)

{% include toc.html %}

## Learning Goals

In this lecture, we discuss *concurrency*.  By the end of this lecture, you should be able to:

- Describe the differences between single-tasking and multi-tasking
- Identify challenges introduced by adding concurrency
- Describe an example of a multitasking system (perhaps with a diagram)
- Identify whether or not a program will benefit from concurrency, either in terms of efficiency or scalability
- Describe various methods of implementing multitasking, and list some advantages/disadvantages of each
- Define and describe time-slicing and how it differs from pseudo-multitasking

## Single-Task versus Multi-Task

Most programs you have written so far will be single tasking systems: there is just one program that runs code *sequentially* from start to finish.  These programs are somewhat easy to design, debug, and test, since the order that your commands are run is deterministic.  For example, let's say you want to read some data, modify it, then write it back to memory,
```cpp
int x = readMemoryValue(shared_memory_address);   // read a value from some memory
x = x+1;                                          // change the value
writeMemoryValue(shared_memory_address, x);       // write the value to some memory
```
You would expect to be able to predict exactly what this program does based on the order of the commands.

With *multi-tasking*, or *concurrency*, the system is decomposed into a set of units -- sections of code (*threads*), programs (*processes*), or a combination -- that can be executed in *parallel*.  These units may be completely independent of each other, or may require some communication so they can coordinate. The functionality of the system as a whole is then distributed between the various units.

**Advantages:**

- Greater *flexibility* -- the system is no longer constrained to run on a single machine, but can be distributed across several servers
- Greater *scalability* -- a single program can be written once and *instatiated* as many times as needed to match a number of resources or to optimize performance

**Disadvantages:**

- Challenges in *decomposing* the system into appropriate concurrently executing threads or processes
- Challenges in *communication* between parallel units
- Challenges in *synchronization* between parallel units
- Challenges in *debugging* and *testing*, since order of commands executing in parallel is not necessarily guaranteed

As an example of a multi-tasking system, consider a web-server providing a service, such as [Google Search](https://www.google.ca).  Every time a client runs a search -- let's say for the term [*concurrency*](https://www.google.ca/search?q=concurrency) -- the server needs to look up some results from a database and present those back to the client.  Imagine if this system was running as a single-tasking process: before Google responded to your search query, it would have to run through and respond to every other query made around the world before yours.  Google receives about 60,000 searches per second, and each query takes about 0.5s to process, so one second's worth of searches would take over 8 hours to complete.  The list of queries would quickly build up in a single thread, and you would never get your results.  On the other hand, if Google had 60,000 servers, each one running a separate instance of the search service (and perhaps multiple instances per server), then each query could be run in parallel, and everyone could get their response almost immediately.  Each 'instance' is running exactly the same main search program.  With this kind of setup, the number of servers and processes-per-server can be adjusted on-the-fly depending on the number of search queries at the time.  This makes the concurrent solution much more *scalable*.

![multitask web service]({{site.url}}/assets/lectures/concurrency/multitask.png)

*A web service accepts client connections through a dispatcher (or [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing))) which forwards the request to an available server.*

Multi-tasking systems are notoriously more difficult to design and debug.  Let's say we took our previous code example, and duplicated it so it ran on two machines simultaneously but shared the same memory at `shared_memory_address`.

<table>
<tr><th>Processor 1</th><th>Processor 2</th></tr>
<tr>
  <td><code>int x = readMemoryValue(shared_memory_address);</code></td>
  <td></td>
</tr>
<tr>
  <td><code>x = x+1;</code></td>
  <td><code>int x = readMemoryValue(shared_memory_address);</code></td>
</tr>
<tr>
  <td><code>writeMemoryValue(shared_memory_address, x);</code></td>
  <td><code>x = x+1;</code></td>
</tr>
<tr>
  <td></td>
  <td><code>writeMemoryValue(shared_memory_address, x);</code></td>
</tr>
</table>

Let's say the value in shared memory started at `0`.  What is the final value of the data in shared memory?  Since two processors run code to increment the value, we might expect the final result to be `2`.  Let's step through it.  The first processor reads the initial value of `0`.  While Processor 1 increments `x`, Processor 2 will still read a `0` since nothing has changed the shared memory yet. The first processor will write the incremented value to memory, while the second processor increments its local copy to `x=1`.  Finally, Processor 2 will then overwrite the value in memory again with `1`.  If the second processor had been delayed slightly before reading the value in shared memory, it may have read the updated value of `1` before incrementing it and saving the data, leading to a final value of `2`.

This type of condition, where the outcome depends on execution timings, is called a `race condition` (we will talk a lot about them in future lectures).  Adding any breakpoints or print statements in the code may change the timings ever-so-slightly-enough that the bug is no longer reproducible.  This is what makes debugging a challenge.  Race conditions, and other new potential bugs that can be introduced when dealing with concurrent systems, are best to avoid through careful program design rather than the test-and-debug approach.  We will discuss approaches to avoiding such bugs when we cover *synchronization*.

## Parallel Programming

Apart from greater scalability and flexibility, allowing things to run in parallel has the *potential* to make your software faster.  *Parallel programming* is the specialized area of concurrent programming that focuses on designing faster algorithms for multi-core or multi-processor systems.

Consider the task of sorting a deck of shuffled cards.  One way to sort them is to divide the deck into four smaller piles, sort each pile, then merge the piles together in a sorted order (a variation of [merge-sort](https://en.wikipedia.org/wiki/Merge_sort)).  As a single person, when it comes to sorting each of the smaller piles, you would sort them sequentially, one after the other.  However, if you had three friends, you could each take one of the smaller piles and sort them in parallel (see [parallel merge-sort](https://en.wikipedia.org/wiki/Merge_sort#Parallel_merge_sort)).  This *should* allow you to sort the entire deck a bit faster than if you were alone.

In general, splitting a system into smaller parallel units introduces *data dependencies*, and the need to be able to *wait* for some of the tasks to complete before continuing.  For example, while sorting our deck of cards, the 'merge' step of the merge-sort cannot begin until we have at least two sorted piles.  This *synchronization* often limits how much of a gain we see by parallelizing.  For some problems, the synchronization may be so limiting or introduce so much overhead, that the parallel version would actually run *slower*.  For example, consider Fibonacci's sequence:

$$F_{n+1} = F_{n} + F_{n-1}, \quad F_0 = F_1 = 1$$

Each term in the sequence depends on the previous two values, so would have to wait until they were computed before continuing.  There would be no point in trying to compute $$F_n$$ and $$F_{n-1}$$ in parallel, since $$F_n$$ depends on $$F_{n-1}$$.

To measure the performance increase of a parallel algorithm running on multiple cores/processes versus the equivalent sequential algorithm on a single core, the [*speedup*](https://en.wikipedia.org/wiki/Speedup) factor is often computed:

$$S_p = \frac{T_1}{T_p},$$

where $$p$$ is the number of processing cores, $$T_1$$ is the time taken of the sequential algorithm running on a single core, and $$T_p$$ the parellel algorithm running on $$p$$ cores.

## Implementing Multi-Tasking

There are a variety of ways we can actually implementing multi-tasking:

- Pseudo multitasking
- Multiple dedicated processors/cores
- Distributed systems
- Time-sliced processors/cores

### Pseudo Multitasking

Pseudo-multitasking is the 'fake' way of multitasking, where we simply switch between a bunch of tasks fast enough so that it *looks* like we are running them in parallel.
```cpp
int main() {

  // continuously cycle through set of monitors
  //   to give appearance of parallel operation
  while ( deviceIsOn() ) {
     monitorTemperature();
     monitorHumidity();
     monitorAtmosphericPressure();
     updateDisplays();
  }

  return 0;
}
```
This is what people usually mean when they say they are "multitasking" colloquially.  They aren't *truly* multitasking, but are trying to make it seem like they are.

- [Why the modern world is bad for your brain](https://www.theguardian.com/science/2015/jan/18/modern-world-bad-for-brain-daniel-j-levitin-organized-mind-information-overload)

For this illusion to work, each of the smaller 'concurrent' tasks must be *brief* enough, and must be able to be called *repetitively*.

**Advantages:**

- Simple to implement and debug since it's still sequential
- No knowledge of the operating system required, since it's just a regular program
- Communication is straightforward, since we can use global variables
- Synchronization is straightforward, since only one task is ever actually running at a time

**Disadvantages:**

- Tasks *must* be brief, and preferably with guaranteed maximum execution times
- Tasks cannot ever loop on a condition or wait for user input, since this would prevent any of the other tasks from executing
- If any task crashes, it brings down the whole system

### Multiple Dedicated Processors

With multiple dedicated processors or cores, each processor/core is given one of the tasks to run on its own.  This is a *true* multi-tasking system: each can run independently (apart from synchronization requirements) and in parallel with the others.  The processors communicate by sharing data through a *shared backplane* such as a [VMEbus](https://en.wikipedia.org/wiki/VMEbus).  If the number of sub-tasks is *fixed* and *known* at *design time*, then a custom system can be built specific to the application.

**Advantages:**

- Can be customized to the application
- Extremely high performance

**Disadvantages:**

- Highly expensive
- Requires complex hardware
- Inflexible, cannot accomodate dynamic process creation (adding another process requires adding another CPU card)
- Difficult to debug

### Distributed Systems

In a distributed system, each task is still assigned to a dedicated processor or core, but these processors can be in different machines/locations and communicate over a network (e.g. by [Ethernet](https://en.wikipedia.org/wiki/Ethernet) or [CAN bus](https://en.wikipedia.org/wiki/CAN_bus)).  Distributed systems are often quite flexible, with the ability to add or remove machines as desired.

To accomodate communication and synchronization, there is often a shared communication channel and messaging protocol based on *token passing*: a node can transmit a message (or packet of information) only when it has possession of a token, can only transmit a single packet, and then must release the token for use by another node.  If it has more information to send, the node must wait to receive the token again.  There is only one token per communication channel so that nodes don't interfere with each other.  You can think of this like having multiple people on a single phone call with a moderator that determines who is allowed to speak, and you are only allowed to say one sentence at a time before giving up your turn.  You can easily add or remove people from the call.  As long as you only speak while it's your turn, nobody will be talking over each other.

In a *deterministic* network, the packet size is fixed, so we can compute the maximum amount of time required to send a complete message based on: the speed of the network (in bits/second), the size of a packet (bytes), the size of a message (number of packets), and the number of nodes in the network.

**Advantages:**

- Flexibility in adding/removing resources
- If one node goes down, the rest of the system can continue

**Disadvantages:**

- Communication over a network is much slower and less reliable than over a bus on a shared backplane
- Complexity in design, often requires redundancy in case a node is disconnected and error correction

### Time-Slicing

This is the more practical and cheaper approach to multitasking, and is used in most systems today.  It makes use of a *Multi-Tasking Operating System (MTOS)* in conjunction with a time-sliced CPU/core.

With time-slicing, the processor or core's available computation time is divided into packets called *time slices*.  The operating system then assigns these slices to programs or threads that need them.  To create the illusion of concurrency, the processor swaps between programs/threads rapidly, executing one time slice after the other sequentially.

Similar to *pseudo-multitasking*, time-slicing only gives the *illusion* of concurrency.  However, it differs in a few key ways:

- splitting the program's execution into smaller chunks is handled automatically by the operating system, not the programmer
- there is no requirement to design the program with repetitively called sub-tasks
- if one program is stuck waiting for a condition or user input, other programs can continue to run

Time-slicing is usually implemented with the aid of a *real-time clock (RTC)* which generates an interrupt signal at short time increments (usually between 10ms and 100ms, depending on process priority and hardware settings).  The interrupt causes the CPU to pause the currently running process/thread and resume another.  This is essentially *time-division multiplexing*.  The switching between tasks is referred to as *context switching*.

Time-slicing addresses all of the main disadvantages of pseudo-multitasking.  It does introduce one new disadvantage, however.  Every time the OS switches between tasks, it needs to save the current task's processor state and load the state for the next one.  This adds *some* overhead, leading to reduced efficiency.  Still, the additional overhead is often a small price to pay for the increased flexibility.  This is what allows your computer to run so many programs at once, seemingly in parallel, without the programs needing to explicitly coordinate with one another.

## Summary

Single tasking is sequential, whereas multi-tasking runs parts of the program in parallel. Concurrent programming is *not* generally focussed on making programs run faster, but more often on designing more flexible and scalable solutions. The field of making algorithms faster by taking advantage of parallelism is called *parallel programming*.

Introducing concurrency often introduces the need for additional communication and synchronization mechanisms.  This not only complicates system design, but also makes programs harder to test and debug.

There are several ways to implement multitasking.  The simplest approach is to manually switch between short repetitive tasks (pseudo-multitasking).  In a true multi-tasking implementation, multiple processors or cores execute code in parallel.  To synchronize their actions, they need to communicate either through hard-wired connections like a shared backplane, or over a shared network communication channel.  Modern CPUs use a technique called time-slicing that allows a single processor or core to execute several programs seemingly in parallel by switching between them rapidly.  This differs from pseudo-multitasking in that the switch is occuring at the operating-system level rather than the program level, which adds greater flexibility.

Modern concurrent systems often take advantage of several of the multitasking implementation techniques.  For instance, most computers now have multiple processors and cores capable of time-slicing, allowing portions of code to be truly executed in parallel while at the same time constantly switching between waiting threads.  And with the explosive growth of web services and cloud computing, we are seeing more and more systems using a distributed approach (again with multi-core, multi-process, time-slicing servers).


## Additional Resources

- [Tutorial on parallel computing (https://computing.llnl.gov/tutorials/parallel_comp )](https://computing.llnl.gov/tutorials/parallel_comp)
- [Computer cluster (http://en.wikipedia.org/wiki/Computer_cluster)](http://en.wikipedia.org/wiki/Computer_cluster)
- [Grid Computing (http://en.wikipedia.org/wiki/Grid_computing)](http://en.wikipedia.org/wiki/Grid_computing) 
