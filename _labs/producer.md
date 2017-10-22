---
layout: lab
title:  Lab 6 - Producers and Consumers
date:   2017-10-19 12:00:00
authors: [C. Antonio Sánchez]
categories: [labs, producers, consumers, monitors, semaphores, condition variables]

---

# Lab 6 -- Producers and Consumers
{:.no_toc}

In this lab we will get some practice with synchronization using semaphores and condition variables, as well as introduce the classic Producer-Consumer pattern
for optimizing the distribution of work between threads.  Our application is a
restaurant ordering system, allowing customers to directly place orders without
needing to wait for the waitstaff.

Much of the code is provided for this lab, available on GitHub [here (https://github.com/cpen333/lab6)](https://github.com/cpen333/lab6).  We will only be modifying small sections of the application to get our order queue and synchronization working.

Feel free to discuss approaches and solutions with your classmates, but labs are to be completed individually.  Each student is expected to be able to answer questions about the content, describe their work, and reproduce their code (or parts thereof).

{% include toc.html %}

## System Overview

When you go to a restaurant, are you ever frustrated with having to wait for someone to come and take your order?  Wouldn't it be more convenient if you could just order your meal whenever you're ready and have it delivered to your table to when it's done?

We are going to design an automated ordering and notification system for a restaurant.  In this new system:
  - Each customer receives a tablet running an application that would allow them to place orders
  - Each order is placed in an *order* queue ready to be made by one of the chefs
  - Each chef has an interface (a mounted tablet or interactive display) that allows them to retrieve the next order in the queue
  - When the meal is ready, the chef places it in a separate queue ready to be served to the customers and notifies the servers
  - Each server has a beeper or app on their phone notifying them that the next meal is ready to be served
  - The server grabs the next meal from the *serve* queue, finds the customer it belongs to, and delivers the meal

![overview]({{site.url}}/assets/labs/producer/overview.jpg)

Rather than create the full restaurant ordering system, we will develop a proof-of-concept using multiple threads within a single process.  Each customer, chef, and server will be running as a separate thread *object*.  The *order* and *serve* queues will be shared queues for synchronously storing order information.

### Source Files

The main system code, found in the `src/` directory is divided into several files:
- `Order.h`: class definition and implementation of an `Order`
- `Chef.h`: class definition and implementation of the Chef module
- `Customer.h`: class definition and implementation of the Customer module
- `Server.h`: class definition and implementation of the Server module
- `Menu.h`: a class storing restaurant menu information for customers to order from
- `OrderQueue.h`: interface for various *queue* implementations to hold a list of `Order`s
- `SingleOrderQueue.h`: simple synchronous "queue" that only stores one order at a time
- `CircularOrderQueue.h`: synchronous queue that stores orders in a fixed-size circular buffer
- `DynamicOrderQueue.h`: synchronous queue that stores orders in a variable-size storage container
- `safe_printf.h`: function definition to allow synchronous `printf` statements
- `restaurant.cpp`: main process

Skim through the files to get a rough idea of what functionality is available.  You do not need to understand all the details of what is involved, only enough to get your bearings for when we modify parts of the program to make it functional.

## The Producer-Consumer Pattern

The producer-consumer pattern is ideal for situations where a task needs to be repeated over and over again, but perhaps with different inputs.  The idea is simple:
- producers package each new task's arguments and place them into a shared queue to be executed
- consumers grab the next set of arguments from the queue and execute the task
This allows us to optimize the distribution of work between threads: as long as there is work to be done, no consumer thread will be waiting idle.

![producer consumer]({{site.url}}/assets/labs/producer/producer_consumer.png)

All synchronization required to allow multiple threads to access/modify the queue simultaneously is handled within the queue itself, internally.  This leads to the [*Producer-Consumer Problem*:](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem) given a fixed-size buffer being used as a queue, synchronize modifications so that a producer won't try to add data into a full buffer, and so that a consumer won't try to remove data from an empty buffer.

### Single element buffer

The simplest solution to the producer-consumer problem is to have a buffer of size one (i.e. a single element), and two semaphores: one for the producer to control writing to the element when it is "empty", and one for the consumer to control reading from the element when it is "full".  On initialization, the producer semaphore should be given a value of one to indicate that there is one slot available for writing, and the consumer semaphore should be set to zero to indicate that there is not yet anything to read.

When adding to the queue, the producer first waits until the slot is free by waiting on the producer semaphore.  Once notified, it is free to add the next item to the queue, then notifies a waiting consumer that the queue is full.  When removing from the queue, the consumer first waits until the slot is full by waiting on the consumer semaphore.  Once notified, it removes the item, then notifies a waiting producer that that the queue is empty.
<table>
  <thead style="text-align: center;">
    <tr><th>Add</th><th>Remove</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>producer.wait();
buff = next;
consumer.notify();</code></pre>
        </div>
      </td>
      <td>
        <div class="highlighter-rouge">
        <pre class="highlight"><code>consumer.wait();
next = buff;
producer.notify()</code></pre>
        </div>
      </td>
    </tr>
  </tbody>
</table>

The two semaphores provide the synchronization to ensure that no two producers or consumers collide.

### Circular buffer

We can extend the single-element solution to instead use a circular buffer.  This would allow more producers to simultaneously add to the queue without blocking.  As long as producers and consumers are adding/removing items at roughly the same rate, this can lead to a faster throughput.

In using the circular buffer, we need to keep track of the next position to write to, as well as the next position to read from.  This introduces two index variables: one for the next producer, and one for the next consumer.  

Since there is more than one slot, it may be possible that two producers/consumers attempt to add/remove an item to/from the queue at the same time, which could lead to a race condition in acquiring the next index.  Therefore we need to protect access to the indices using mutual exclusion.  This introduces two mutexes: one for the producer index, and one for the consumer index.  Once we have the appropriate index, the actual copying of data can be done outside of the lock since we know we are the only one accessing this particular slot due to the combination of the semaphore and mutex.  This can be important, since for large data structures the memory copying can be the bottleneck of the add/remove operation.

### Dynamically sized queue

If producers tend to add items to the queue in bursts and the circular buffer is not large enough to accommodate them, the producers may still be prone to blocking.  We can try to fix this by using a dynamically sized queue rather than a circular buffer.  If internally the queue will always adjust its size to account for incoming data, we can guarantee that producers will *never* block, and consumers will only block *until* the queue is non-empty.

In C\+\+, dynamic queues can be implemented using the `std::deque` class found in the `<deque>` header.  A *deque* is a double-ended queue, allowing us to add or remove items from the front or back.  When using as a queue, we always add to the back and remove from the front.

<table>
  <thead style="text-align: center;">
    <tr><th>Add</th><th>Remove</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <div class="highlighter-rouge">
          <pre class="highlight"><code>deque.push_back(next);</code></pre>
        </div>
      </td>
      <td>
        <div class="highlighter-rouge">
        <pre class="highlight"><code>next = deque.front();  // get front
deque.pop_front();     // remove front</code></pre>
        </div>
      </td>
    </tr>
  </tbody>
</table>

None of these C\+\+ containers in the standard template library are thread-safe,
so we will need to protect access while adding/removing elements using mutual exclusion.

We still need a mechanism for causing consumers to block if the queue is empty.  For this
we can either use a semaphore or a condition variable.  In this case it may be slightly
more convenient to use a condition variable, since we will need to lock a mutex anyways when removing an item from the deque.

## Restaurant

The new restaurant ordering system will use the producer-consumer pattern with two synchronous queues: one to store new orders from customers, and one to store completed orders ready to serve.  

The restaurant has a `Menu` that lists all items that a customer is able to order.  Each menu item has a unique ID that can be used to identify it.

When customers arrive, they are given a unique ID.  This allows the system to keep track of their orders, and can be used by the servers to identify the customer when their order is ready.

An order consists of two integers: `{customer_id, item_id}`.  The customer ID refers to the customer who placed the order, and the item ID is the menu item they ordered.

For new orders, the customers act as producers, generating new orders and placing them in the *order* queue.  The chefs act as consumers, removing orders from the queue and preparing the meals.  For completed meals, the chef processes act as producers, placing the order in the *serve* queue.  The servers act as consumers, removing orders from the queue and delivering them to the customer.

### Task 1: Order queues

All synchronization is handled inside the shared queues representing orders and meals ready to serve.  Your first task is to implement three types of queues:
- `SingleOrderQueue`: a synchronous queue with a single-element internal buffer
- `CircularOrderQueue`: a synchronous queue with a fixed-length internal circular buffer
- `DynamicOrderQueue`: a synchronous queue with a dynamically-sized internal buffer

The class diagram for these is as follows:
![order queue class diagram]({{site.url}}/assets/labs/producer/order_queue.png)

Try running the system using each of these queues.  Do you notice any differences?  Which do you think makes the most sense in terms of the restaurant system, and why?

### Task 2: Serve notifications

If you run the system as-is, you may have noticed that customers do not necessarily wait until their meals are served before leaving.  Your next task is to fix this by adding a synchronization mechanism within `Customer`. Add whichever synchronization mechanism makes the most sense to cause each customer to wait for their orders and eat them before leaving.  Note that when a `Server` delivers a meal, it calls the `Customer::serve(Order& order)` method.

### Task 3: Poison pills

After all customers leave, the chefs and servers will still be waiting on the order and serve queues for the next item which will never come.  We need to introduce a mechanism to wake them up and inform them they are done for the day.

Since the consumer may potentially be stuck waiting *inside* the queue's `get()` method, the only way we can wake it up is by sending a new item into the queue.  To prevent the item from being processed as usual, however, the item must somehow indicate that it is not a valid entry, but instead is being used to notify the consumer to close.  We call this special indicator item a *poison pill*.  When a consumer retrieves a poison pill from the queue, it should close down and die.

Your final task is to create and use poison pills to allow every thread to exit gracefully at the end of the program.
1. Define a special `Order` to use as a poison pill
2. Modify both `Chef` and `Server` to stop their main loops when they receive a poison pill
3. At the end of `restaurant.cpp`, send an appropriate number of poison pills into the queues to trigger all chefs and servers to quit
