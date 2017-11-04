---
layout: exam
title:  Midterm
date:   2017-11-03 21:00:00
authors: [C. Antonio Sánchez]
categories: [exam, mutex, condition variable, testing]
usemath: true

---

# Midterm
{:.no_toc}

Welcome to the CPEN 333 -- System Software Engineering Take-Home Midterm!  Please take the time to read all the instructions and the questions carefully.  

Before beginning, you will be asked to read and sign a [Take-Home Midterm Agreement Form]({{site.url}}/assets/exams/midterm/MidtermAgreement.pdf).  This signed form **must be included** with your submission for you to receive a grade.  

Submissions will be accepted through UBC Connect.  The deadline listed on Connect is final.  You will have at least 48 hours from the time the midterm is posted to complete and submit your exam solutions.  Late entries will not be graded.

After the midterm, there is an optional exit survey to let us know what you thought about the midterm: was it too hard? too long? too wordy? too fun?  This is for us to get your perspective so we have more context when grading your submissions.

Good luck!

{% include toc.html %}

## General Instructions

This is an open-book, open-note, open-IDE, open-internet midterm exam.  You may reference any *existing* materials or documentation you find anywhere, including course notes, C\+\+ class or method specifications, web forums, and your own lab solutions.

You **may not**, however, discuss problems **or *any* material related to the course** with *anyone* during the period of the exam.  This includes classmates, teaching assistants, instructors, friends, enemies, parents, siblings, or long lost cousins.  You may not ask material-related questions on Piazza, Stack Overflow, Reddit, or any other digital, analog, or live forum.

Before you begin, you **must** read, initial and sign the [Take-Home Midterm Agreement Form ({{site.url}}/assets/exams/midterm/MidtermAgreement.pdf)]({{site.url}}/assets/exams/midterm/MidtermAgreement.pdf).  This signed form **must be included** with your submission for you to receive a grade.

Much of the code is provided for you for this midterm, available on GitHub [here (https://github.com/cpen333/midterm)](https://github.com/cpen333/midterm).

There are four main problems to complete, each with several tasks.  Similar to the labs, most of the code is already provided for you.  Unless explicitly stated otherwise, you are free to change *any* of the code if it will help you to solve the problem at hand.  You should document any changes using comments in your code.

If any question or task is unclear, make assumptions that will help you solve the problem, mark them down as comments in your code, and continue with your solution.

While you are not explicitly being graded on coding style, anything you do to help with the clarity of your code will go a long way to helping us understand your intentions.  This includes proper indentation to indicate scope, meaningful variable and function names, and comments on non-trivial sections of code to explain your approach.

Part marks will be awarded as long as we understand your intentions.  If you're not entirely sure how to implement something, or can't quite get part of your code to work correctly, write down your approach in the form of comments.  If you identify a bug in your own code but don't know how to fix it, write down what you *think* is happening and how you might solve it given more time.

## Question 1 -- Spin-Lock

A *spin-lock* continuously checks a flag to see when a resource becomes free, then sets the flag in order to *acquire* the lock and resource.  This check-and-set operation must be done in an atomic manner to avoid race conditions with other threads.  In C\+\+, we can use an `std::atomic_flag`, found in the `<atomic>` library for this check-and-set.

We will first create a spin-lock class that can be used to enforce mutual exclusion.  We will then implement a small program to test our lock.

The code for this question can be found in the `q1_spinlock` folder. There are only two source files:
- `SpinLock.h`: implementation of the desired `SpinLock`
- `main.cpp`: main program for testing the lock

### **Task 1: Spin-Lock [5 marks]**

In the file `SpinLock.h`, you are given a template for creating a `SpinLock` class.  The class supports three operations:
- `lock()`: *acquires* the lock, blocking the current thread if necessary until the lock becomes available
- `try_lock()`: tries to acquire the lock, returning true if the lock is acquired successfully, false if it is already locked
- `unlock()`: releases the lock, regardless of whether or not the current thread is the one who locked it

Complete the implementation details.

```cpp
#include <atomic>
#include <thread>

class SpinLock {
  std::atomic_flag flag_;

 public:

  SpinLock() : flag_(ATOMIC_FLAG_INIT) {}

  /**
   * Acquires the lock, blocking if necessary
   */
  void lock() {
    //=======================================
    // TODO: Implement lock operation
    //=======================================
  }

  /**
   * Tries to acquire the lock
   * @return true if lock is available and acquired successfully, false otherwise
   */
  bool try_lock() {
    //=======================================
    // TODO: Implement try_lock operation
    //=======================================
    return false;
  }

  /**
   * Releases or unlocks the lock, regardless of who may have locked it
   */
  void unlock() {
    //=======================================
    // TODO: Implement unlock operation
    //=======================================
  }
};
```

### **Task 2: Multithreading [5 marks]**

The main source code for testing our spin-lock can be found in `q1_spinlock/src/main.cpp`.  The method `thread_increment` increments a shared counter variable one-by-one 10 million times.

```cpp
void thread_increment(long& counter) {
  //==============================================
  // TODO: SAFELY increment by one at a time
  //==============================================
  for (int i=0; i<10000000; ++i) {
    ++counter;
    // dangerous_increment(counter);
  }
}
```

In the main method, we currently call this function 5 times sequentially.

```cpp
int main() {

  long counter = 0;

  //==========================================================
  // TODO: create and run 5 threads calling thread_increment
  //==========================================================
  for (int i=0; i<5; ++i) {
    thread_increment(counter);
  }
  std::cout << "Counter value: " << counter << std::endl;

  return 0;
}
```

1. Modify the code so that instead of 5 sequential calls, we create 5 threads and run them concurrently.
2. Use your `SpinLock` from **Task 1** to protect the critical section in the `thread_increment` function.  You can "pass-in" your spin-lock however you like, modifying any function definitions if necessary.

### **Task 3: Exception-Safety [5 marks]**

The `dangerous_increment(long& counter)` method increments a counter by 1, but will periodically throw an exception.

```cpp
void dangerous_increment(long& counter) {

  // randomly throw an exception
  static auto rand = std::default_random_engine((int)std::chrono::system_clock::now().time_since_epoch().count());
  static auto unif = std::uniform_real_distribution<double>(0, 1);
  if (unif(rand) < 0.00000001) {
    throw std::exception();
  }

  // increment
  ++counter;
}
```

1. In the `thread_increment` method, replace `++counter` with `dangerous_increment(counter)`.
2. Modify your code to ensure your spin-lock is unlocked in the case of an exception.

## Question 2 -- Primes

A prime number is a *positive integer* that has exactly *two divisors*: 1 and itself.  The first 10 prime numbers are: 2, 3, 5, 7, 11, 13, 17, 19, 23 and 29.

We are going to write and test a method to check whether any number in the range [1, $$2^{31}-1$$] is prime (i.e. will work for any positive 4-byte integer).

One of the fastest methods for checking whether or not a number is prime is to store a pre-generated list of prime numbers and check if a given number is contained in this list.  Unfortunately, there are 105,097,565 prime numbers less than $$2^{31}$$, which would take about 401 MB of memory.  We can get away with a much smaller list by combining this approach with trial-division.  If a number $$N$$ is *not* prime, then it must have a prime divisor less than or equal to $$\sqrt{N}$$.  For $$N = 2^{31}-1$$, this means we only need a list of primes less than 46341, of which there are 4792 (19 KB of memory).

Therefore we can check if a number $$N$$ is prime by doing the following:
- generate a list of primes less than 46341
- if $$N < 46341$$ search our list of primes
   - if $$N$$ is found within this list then it is prime, otherwise it is not
- if $$N \geq 46341$$ check the remainder of $$N/p$$ for every prime $$p\leq\sqrt{N}$$
   - if any remainder is zero, $$N$$ is not prime
   - if all remainders are non-zero, $$N$$ is prime

Some programmers have already implemented most of the required methods.  However, they forgot to document one of the major methods, and they couldn't quite get the prime-checking to work correctly.  We will use unit-testing to try to uncover and fix any bugs, and complete any missing function specifications.

The code for this question can be found in the `q2_primes` folder.
- `primes.h`: header file for prime-checking methods
- `primes.cpp`: implementation of prime-checking methods
- `TestException.h`: a simple exception class
- `primes_test.cpp`: unit testing methods

For the unit-test implementations, you may choose to use your own unit-testing framework in place of `primes_test.cpp` if you wish.  Make sure to document your approach using comments.

### **Task 1: Specifications [5 marks]**

The `primes.h` header contains three function declarations, two of which are documented:

```cpp
//===================================================================
// TODO: Write specification for generate_primes
//===================================================================
std::vector<int> generate_primes(int n);

/**
 * Finds a value within the supplied vector of integers
 * @param data vector to search
 * @param val value to find
 * @return the index of the FIRST occurrence of the value val in data
 *         or -1 if not found
 */
int binary_search(const std::vector<int>& data, int val);

/**
 * Checks if a positive integer is prime
 * @param n integer to check, must satisfy 0 < n <= 2^31-1
 * @return true if number is prime, false otherwise
 */
bool is_prime(int n);
```

In the `primes_test.cpp` file there is a method defined to help with testing `generate_primes(...)`:

```cpp
/**
 * Unit Tests for generate_primes
 * @throws TestException if a unit test fails
 */
void test_generate_primes() {

  //===================================================================
  // TODO: Test generate_primes to figure out what exactly it does
  //===================================================================
  test_generate_primes(10, {2, 3, 5, 7});
}
```

By carefully selecting a set of tests and seeing what is returned, try to figure out exactly what `generate_primes(...)` does for a given set of inputs.  

Your task:
1. Write a few tests to determine the set of valid inputs and corresponding outputs
2. Use this information to write a full function specification in `primes.h`

### **Task 2: Binary Search [5 marks]**

You begin to suspect that there may be a few bugs in the `binary_search(...)` method.  To find and fix them all, you need to write unit tests for the function.  

1. Inside `primes_test.cpp`, write a complete set of unit tests for `binary_search(...)`.  Document your test strategy using comments inside the method.

   ```cpp
   /**
    * Unit Tests for binary_search
    * @throws TestException if a test fails
    */
   void test_binary_search() {

     //===================================================================
     // TODO: Implement unit tests for binary_search
     //===================================================================

     // single element found
     test_binary_search({0, 1, 2, 3, 4, 5}, 2, 2);
   }
   ```
   2. In `primes.cpp`, fix each of the errors in the `binary_search(...)` function as you encounter them.

### **Task 3: Checking Primes [5 marks]**

Unfortunately, after further testing, the `is_prime(...)` function still doesn't seem to always work correctly.  Now that you are confident in `generate_primes(...)` and `binary_search`, you have narrowed-down any potential bugs to the `is_prime(...)` function itself.

1. Inside `primes_test.cpp`, write a complete set of unit tests for `is_prime(...)`.  Document your test strategy using comments inside the method.

   ```cpp
   /**
    * Unit tests for is_prime
    * @throws TestException if a unit test fails
    */
   void test_is_prime() {

     //===================================================================
     // TODO: Implement unit tests for is_prime
     //===================================================================

     // small prime
     test_is_prime(11, true);
   }
   ```
2. In `primes.cpp`, fix each of the errors in the `is_prime(...)` function as you encounter them.

## Question 3 -- Ninja Turtles

Leonardo, Donatello, Michelangelo and Raphael are crime-fighting mutant turtles living in New York City.  The brothers always work as a team: whenever one is defeated, one of his brothers picks him back up again.  The evil Shredder is the turtles' arch-enemy, determined to defeat them all.

Raphael is currently off on his own, leaving a three-turtle team to start.

We will simulate the crime-fighting action using a multi-process system.  Each character is represented by a separate process.  When one process is closed, that character is "defeated".  To pick a character back up again, the process must be restarted in *detached* mode.

The starting code for this question can be found in the `q3_turtles` folder.
- `turtles.h`: main header file containing information to be shared between all turtles
- `leonardo.cpp`: implementation of Leonardo's process
- `donatello.cpp`: implementation of Donatello's process
- `michelangelo.cpp`: implementation of Michelangelo's process
- `raphael.cpp`: implementation of Raphael's process
- `shedder.cpp`: implementation of Shredder's process

**Note:** to manually terminate a process on a Mac, you can open the Terminal app and issue the `killall <name>` command.  For example,

```
killall michelangelo
```

will terminate Michelangelo.

### **Task 1 -- Fix the Flaws [5 marks]**

As written, when Leonardo is terminated, Michelangelo starts him back up.  When Donatello is terminated, Leonardo starts him back up.  When Michelangelo is terminated, Donatello starts him back up.  

![turtle cycle]({{site.url}}/assets/exams/midterm/turtle_cycle.png)

They accomplish this using a *check-in* system.  At a regular intervals, each turtle increments a *check-in counter* that the other processes can read.  If a turtle fails to check in regularly, he is presumed to be *defeated*, so the appropriate brother launches another instance of the process.

However, there are a few potential flaws in the current design related to memory access and race conditions.  Identify them with comments in your code, and modify the code to fix them.

### **Task 2 -- Bring in Raphael [5 marks]**

Raphael wants to join the action.  Implement the body of the process, and loop him in to create a four-turtle team.  You will need to modify one of the other turtles to complete the cycle.

### **Task 3 -- Enter the Shredder [10 marks]**

The evil Shredder now joins the mix.  Shredder can decide which turtle to attack and defeat.  When a turtle is attacked, he should shut down within a reasonable amount of time.  An attacked turtle is not able to launch another turtle.

You job is to implement the mechanism to trigger a defeat.  Make sure you account for any potential race conditions.

1. Modify the shared memory layout in `turtles.h` to include appropriate variables for triggering this *defeat*
2. Modify each of the turtles to respond to the trigger
3. Complete the implementation of Shredder

## Question 4 -- Roller Coaster Ride

A roller coaster has a single car that can hold *up to four* passengers.  Your task is to design the synchronization mechanism that satisfies the following conditions:
1. Only 4 passengers can get on the roller coaster at a time (i.e. they wait their turn while people are boarding)
2. The roller coaster will wait to start until *either*
  - four passengers have boarded
  - at least one passenger has boarded and there are no other passengers waiting in line
3. Each passenger will wait until the ride is over before getting off
4. New passengers will wait until all previous passengers are off before boarding

You may use whichever synchronization primitives you wish to solve this problem.

Starter code for this question can be found in the `q4_rollercoaster` folder.
- `common.h`: header file containing information to be shared between all processes
- `passenger.cpp`: Passenger process
- `rollercoaster.cpp`: Roller Coaster process
- `launcher.cpp`: process to launch the roller coaster system.  **This file should not be modified.**

### **Condition 1 -- Boarding [5 marks]**

When the roller coaster is boarding, a maximum of four passengers can enter at a time.  If there is a fifth passenger waiting, she/he must wait for the next ride.

Implement this *boarding* synchronization stage.  Document your approach using comments in your code at the appropriate sections beginning with:

```cpp
// Condition 1:
```

### **Condition 2 -- Starting [10 marks]**

The roller coaster will continue boarding until either it is full with four passengers **[5 marks]**, or until there is at least one passenger on board and no more passengers waiting **[5 marks]**.

Implement this *starting* synchronization stage.  Document your approach using comments in your code at the appropriate sections beginning with:

```cpp
// Condition 2:
```

**Note:** if you only get the roller coaster working when it is full with 4 passengers (and not the *up to* part), you can still receive the marks for that portion.

### **Condition 3 -- Riding [5 marks]**

A passenger must remain on the ride until the roller coaster has come to a complete stop and allows her/him to exit.

Implement this *riding* synchronization stage.  Document your approach using comments in your code at the appropriate sections beginning with:

```cpp
// Condition 3:
```

### **Condition 4 -- Exiting [5 marks]**

No one is allowed to board until all previous passengers have exited the ride.  Once the final passenger has left, the roller coaster can signal new passengers to begin boarding again.

Implement this *exiting* synchronization stage.  Document your approach using comments in your code at the appropriate sections beginning with:

```cpp
// Condition 4:
```

## Submission

All submissions should be made through UBC Connect ([https://connect.ubc.ca](https://connect.ubc.ca)).  On the main course page, you should see an entry called **Midterm Submission** at the bottom.  

![submission]({{site.url}}/assets/exams/midterm/submission.png)

This page will allow you to attach files and write any comments.  You can ignore the text submission component.

Please gather all source code and project files into a **single zip file** (extension .zip, .tar, or .tar.gz) and attach it to your submission.  If possible, exclude any compiled binaries (such as executables).  You do not need to include the course library, or any other included libraries in your submission.  For help on creating a zip file, see [https://www.wikihow.com/Make-a-Zip-File](https://www.wikihow.com/Make-a-Zip-File).

You may submit your solutions more than once.  However, we will only grade your final attempt.

Please also ensure that you have submitted the signed [Take-Home Midterm Agreement Form ({{site.url}}/assets/exams/midterm/MidtermAgreement.pdf)]({{site.url}}/assets/exams/midterm/MidtermAgreement.pdf).  It can be contained within your zip file, or as a separate attachment.  

If you are having trouble either digitally signing the agreement form, or scanning/taking a picture of a signed physical copy, then you may do the following:
- Create a new text file with a declaration that you have read the form and have adhered to all terms
- Include your name, student #, and email address

## Exit Survey

Congratulations on finishing the take-home midterm for CPEN 333 -- System Software Engineering!

Due to request, we have prepared an exit survey for this take-home midterm.  The purpose of the survey is to help us teaching staff gauge the difficulty of the midterm from *your* perspective.

Participation is entirely optional, and unless you choose to reveal your identity, all responses will be kept completely anonymous, even from us.

Any information you *do* provide may help us to put your collective performance in perspective and to adjust content going forward.

[https://survey.ubc.ca/s/cpen333/midterm/](https://survey.ubc.ca/s/cpen333/midterm/)
