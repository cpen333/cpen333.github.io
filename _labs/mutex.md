---
layout: lab
title:  Lab 3 - Testing and Mutexes
date:   2017-09-28 17:50:00
authors: [C. Antonio Sánchez]
categories: [labs, threads, multithread, shakespeare, testing]

---


# Lab 3 -- Testing and Mutexes
{:.no_toc}

In this lab, we will learn about unit testing, and will get some practice with thread synchronization using mutexes.  First we will create a method that counts the number of words in a string, and test this thoroughly to ensure reliability under a variety of circumstances.  Next, we will use this method to help us count how many words each character has to memorize in a collection of plays by Shakespeare.  We will then switch gears and look at synchronization between multiple processes.

To help get you started, some of the code is posted on GitHub [here (https://github.com/cpen333/lab3)](https://github.com/cpen333/lab3).

Feel free to discuss approaches and solutions with your classmates, but labs are to be completed individually.  Each student is expected to be able to answer questions about the content, describe their work, and reproduce their code (or parts thereof).

{% include toc.html %}

## Part 1 -- Shakespearean Word Counts

### Counting Words and Unit Tests

Consider the function:
```cpp
/**
 * Counts number of words, separated by spaces, in a line.
 * @param line string in which to count words
 * @param start_idx starting index to search for words
 * @return number of words in the line
 */
int word_count(const std::string& line, int start_idx);
```
The method specification tells us that it counts the number of words in a string, with each new word separated by a space from the previous.  The method also allows us to specify a starting index within the string to start searching for words.  Think about how you might test this function to see if it is working correctly.  What are some sample strings you think you should test it with?

Once you've thought a bit about the problem for a few minutes, start the implementation.  We will separate the declaration into a header file named `word_count.h` and write the implementation in a `word_count.cpp` file.

`word_count.h`:
```cpp
#ifndef LAB3_WORD_COUNT_H
#define LAB3_WORD_COUNT_H

#include <string>

/**
 * Counts number of words, separated by spaces, in a line.
 * @param line string in which to count words
 * @param start_idx starting index to search for words
 * @return number of words in the line
 */
int word_count(const std::string& line, int start_idx);

#endif //LAB3_WORD_COUNT_H
```

`word_count.cpp`:
```cpp
#include "word_count.h"

// implementation details
int word_count(const std::string& line, int start_idx) {

  // YOUR IMPLEMENTATION HERE

  return 0;
}
```
Note that we do not need to mark `word_count` as `inline` here because we have separated the declaration from the implementation (*the implementation inside `word_count.cpp` will only be compiled once*).

Do you think you've implemented the function correctly?  Now we're going to test it.  Unit tests are usually written in a separate testing source file so that your tests don't clutter your actual code.  You don't want these tests to make it into the production code.  We are going to manually create some unit tests in a file called `word_count_test.cpp`:

```cpp
#include "word_count.h"
#include <exception>
#include <iostream>

// Exception class to throw for unit test failures
class UnitTestException : public std::exception {
  std::string line_;
  int start_idx_;
  int result_;
  int expected_;

 public:

  // constructor collecting all information
  UnitTestException(const std::string& line, int start_idx,
                    int result, int expected) :
    line_(line), start_idx_(start_idx),
    result_(result), expected_(expected) {}

  const char* what() {
    return "Unit test failed";
  }

  std::string info() {
    std::string out;
    out.append("line: ");
    out.append(line_);
    out.append(", start_idx: ");
    out.append(std::to_string(start_idx_));
    out.append(", result: ");
    out.append(std::to_string(result_));
    out.append(", expected: ");
    out.append(std::to_string(expected_));
    return out;
  }

};

/**
 * Tests word_count for the given line and starting index
 * @param line line in which to search for words
 * @param start_idx starting index in line to search for words
 * @param expected expected answer
 * @throws UnitTestException if the test fails
 */
void wc_tester(const std::string& line, int start_idx, int expected) {

  int result = word_count(line, start_idx);

  // if not what we expect, throw an error
  if (result != expected) {
    throw UnitTestException(line, start_idx, result, expected);
  }

}

int main() {

  try {

    // YOUR TESTS HERE

    wc_tester("hello world", 0, 2);

  } catch(UnitTestException &ute) {
    std::cout << ute.info() << std::endl;
  }

}
```

We start by creating an exception class called `UnitTestException` that will be `thrown` whenever one of our tests fail.  This exception stores everything we need to know about the failed test to help us debug.  In the above code, `UnitTestException` captures all the inputs to the `word_count` function, the result, and the expected result.

We then define a *tester* function that takes the inputs along with an expected answer, calls our `word_count` function, then throws the exception if we get something unexpected.

Finally, our main method has a `try - catch` block that will call our tester function with a variety of inputs and expected outputs.  If any tests fail, the `catch` part catches the exception and prints out information about the failed test so we can try to debug our code.

**Your task:** add a variety of tests.  Try to "break" your own code.  It is very rare to have a method that runs perfectly, for all inputs, on a first or second try.  You should have enough tests so that you've covered all expected *behaviours* of your method.  Think about possible partitions of the input space, as well as boundary/edge/corner cases.  In particular, think about:
- different numbers of words in a string
- different starting indices
- different lengths of strings
- extra spaces at the start, end, or between words
- ...?

### Maps

Maps are *incredibly* useful data structures.  Conceptually, they are quite simple: all they do is associate pairs of data together.  Think about an array or vector.  When you set `array[10] = "Steve"`, in some sense you are associating the number 10 with the name *Steve*.  You can later retrieve the name associated with number 10 using the `std::string name = array[10]`.  

Maps allow us to generalize this, associating pretty much any type of data with any other type of data.  For example, we can have a map of strings to numbers using
```cpp
#include <map>
//...
std::map<std::string,int> map;
map["Steve"] = 10;
```
A map *maps* data from a *key* to a *value*.  In the above, the keys are of type `std::string`, and the values are of type `int`.  For `std::map`, the only (default) restriction on keys is that they are comparable with a `<` operator.  If you have a class that does not implement `<` but you still want to use it as a key in a map, you can write your own comparator (we will do this later when it comes to sorting).

The key and value pairs are stored in the map as an `std::pair`.  Pairs have two member variables:
```cpp
std::pair<std::string,int> kv("Tiffany", 786);
std::string key = kv.first;
int val = kv.second;
```
We can add key-value pairs directly to the map using
```cpp
map.insert(std::pair<std::string,int>("Georgia", 15));
// or
map.insert({"Georgia", 15});
```
Keys are considered unique in a map, so if you later assign
```cpp
map["Georgia"] = 16;
```
this will overwrite the original value.

You can check a map to see if a particular key exists using the `find(...)` method:
```cpp
auto it = map.find("Gregory");
if (it == map.end()) {
  // not found
} else {
  // found
  std::string key = it->first;
  int val = it->second;
}
```
The `find(...)` method returns an *iterator*, which you can treat similarly to a pointer.  If the key is not found in the map, the map's `.end()` is returned, otherwise an iterator to the item is returned which lets you access the key-value pair directly.  Iterators can be dereferenced just like pointers, and you can access their members using the `->` operator just like pointers.

You can iterate through all the entries in a map just like for a vector or any other iterable:
```cpp
for (const auto& pair : map) {
  std::cout << pair.first << "->" << pair.second << std::endl;
}
```
You should find that the map iterates through the keys in increasing order (in our case, alphabetically).

### Sorting

We've already seen how to implement sorting algorithms like QuickSort, but do we want to do this every time from scratch when we need to sort a new data type?  Of course not.  Luckily for us, the `<algorithm>` library is here to help.  We can easily sort a set of strings, for example, by calling
```cpp
#include <algorithm>
//...
std::vector<std::string> names = {"Bob", "Xin", "Mohammad", "Carlos"};

std::sort(names.begin(), names.end());
```
The `sort(...)` method takes in two *iterators* as arguments, and sorts everything in between (including the first but excluding the second).  This works fine for things like strings that have a natural `<` comparison operator, but how do we handle sorting of some other data structures like pairs of custom objects?  Well, we can write a special *comparator* to order them.  Comparators are *things* that accept two instances of your objects and return a boolean that is `true` if the items are in the correct order, or `false` if they should be switched.  You can write a method as a comparator, or a class, or a lambda... anything that has the form
```cpp
bool compare(const type& a, const type& b);
```
For example, we could write a comparator method for strings that would sort in reverse order using
```cpp
bool reverse_compare(const std::string& a, const std::string& b) {
  return a > b;
}
```
and then use this method to sort our list of names in descending order by supplying the comparator:
```cpp
std::sort(names.begin(), names.end(), reverse_compare);
```
The one caveat to writing comparators is that you need to enforce a *unique* ordering.  For example, let's say we want to sort pairs of `{name, number}` by increasing number.  If we write our comparator as
```cpp
bool bad_sort_by_number(const std::pair<std::string,int>& a, const std::pair<std::string,int>& b) {
  return a.second < b.second;
}
```
then the ordering would not be unique.  If two *different* names are paired with the same number -- e.g. `{Dave, 10}`, `{Steve, 10}` -- it would be random which one would come first once sorted.  To enforce a unique ordering, you'll want to do something like
- sort by numbers first
- sort by names second if numbers are equal

### Counting Words in Shakespeare

Shakespeare's characters talk a lot in his plays.  For example, Othello says approximately 12000 words, and Romeo about 10000.  That's a lot of memorization.  What characters say the most in all of Shakespeare's plays?

We already have a word count function that we can make use of.  Now all we need are some files to use it on.

[Project Gutenburg](https://www.gutenberg.org/) offers free digital copies of many literary works for which the copyright has either expired or never existed.  It's a great resource for reading many of the classics.  In the GitHub `data` folder [here](https://github.com/cpen333/lab3/tree/master/data) you will find all of Shakespeare's works.  Note that not all of them are plays though.

If you open one of the plays, such as Romeo and Juliet, you will notice that *most* lines containing dialogue either begin with two spaces or with four spaces.  If the line starts with two, the line contains the name (or short-form) of the character speaking, followed by a period, followed by the character's dialogue.  If the line starts with four, it is usually a continuation of the previous character's lines.  This approach is by no means perfect, but should be sufficient for us to determine which Shakespearean character is the most verbose.

We have provided much of the code for you in this exercise.  We will map *Character* → &copy; *Word Count* in an `std::map`.  We will parse a selection of Shakespeare's plays, extract the speakers of each line of dialogue, count the number of words, and update the counts in the map.  This should all be done in a multithreaded way.

```cpp
#include <map>
#include <string>
#include <mutex>
#include <algorithm>
#include <iostream>
#include <fstream>
#include <vector>
#include <thread>

#include "word_count.h"

/**
 * Checks if the line specifies a character's dialogue, returning
 * the index of the start of the dialogue.  If the
 * line specifies a new character is speaking, then extracts the
 * character's name.
 *
 * Assumptions: (doesn't have to be perfect)
 *     Line that starts with exactly two spaces has
 *       CHARACTER. <dialogue>
 *     Line that starts with exactly four spaces
 *       continues the dialogue of previous character
 *
 * @param line line to check
 * @param character extracted character name if new character,
 *        otherwise leaves character unmodified
 * @return index of start of dialogue if a dialogue line,
 *      -1 if not a dialogue line
 */
int is_dialogue_line(const std::string& line, std::string& character) {

  // new character
  if (line.length() >= 3 && line[0] == ' '
      && line[1] == ' ' && line[2] != ' ') {
    // extract character name

    int start_idx = 2;
    int end_idx = 3;
    while (end_idx <= line.length() && line[end_idx-1] != '.') {
      ++end_idx;
    }

    // no name found
    if (end_idx >= line.length()) {
      return 0;
    }

    // extract character's name
    character = line.substr(start_idx, end_idx-start_idx-1);
    return end_idx;
  }

  // previous character
  if (line.length() >= 5 && line[0] == ' '
      && line[1] == ' ' && line[2] == ' '
      && line[3] == ' ' && line[4] != ' ') {
    // continuation
    return 4;
  }

  return 0;
}

/**
 * Reads a file to count the number of words each actor speaks.
 *
 * @param filename file to open
 * @param mutex mutex for protected access to the shared wcounts map
 * @param wcounts a shared map from character -> word count
 */
void count_character_words(const std::string& filename,
                           std::mutex& mutex,
                           std::map<std::string,int>& wcounts) {

  //===============================================
  //  IMPLEMENT THREAD SAFETY IN THIS METHOD
  //===============================================

  std::string line;  // for storing each line read from the file
  std::ifstream file (filename);

  // read contents of file if open
  if (file.is_open()) {

    std::string character = "";  // empty character to start

    // line by line
    while ( std::getline (file,line) ) {

      int idx = is_dialogue_line(line, character);
      if (idx > 0 && !character.empty()) {

        int nwords = word_count(line, idx);

        // add character if doesn't exist, otherwise increment count

        //=================================================
        // YOUR JOB TO ADD WORD COUNT INFORMATION TO MAP
        //=================================================

      } else {
        character = "";  // reset character
      }

    }
    file.close();  // close file
  }

}

/**
 * Comparator, orders by number decreasing, then by name
 * @param p1 first pair
 * @param p2 second pair
 * @return true if p1 should come before p2, false otherwise
 */
bool wc_greater_than(std::pair<std::string,int>& p1, std::pair<std::string,int>& p2) {

  //===============================================
  // YOUR IMPLEMENTATION HERE TO ORDER p1 AND p2
  //===============================================

  return false;
};

/**
 * Sorts characters in descending order by word count
 *
 * @param wcounts a map of character -> word count
 * @return sorted vector of {character, word count} pairs
 */
std::vector<std::pair<std::string,int>> sort_characters_by_wordcount(
    const std::map<std::string,int>& wcounts) {

  std::vector<std::pair<std::string,int>> out;
  out.reserve(wcounts.size());   // reserve memory for efficiency

  // sort characters by words descending
  for (const auto& pair : wcounts) {
    out.push_back(pair);
  }
  std::sort(out.begin(), out.end(), wc_greater_than);

  return out;
}

int main() {

  // map and mutex for thread safety
  std::mutex mutex;
  std::map<std::string,int> wcounts;

  std::vector<std::string> filenames = {
      "data/shakespeare_antony_cleopatra.txt",
      "data/shakespeare_hamlet.txt",
      "data/shakespeare_julius_caesar.txt",
      "data/shakespeare_king_lear.txt",
      "data/shakespeare_macbeth.txt",
      "data/shakespeare_merchant_of_venice.txt",
      "data/shakespeare_midsummer_nights_dream.txt",
      "data/shakespeare_much_ado.txt",
      "data/shakespeare_othello.txt",
      "data/shakespeare_romeo_and_juliet.txt",
  };

  //============================================================
  // YOUR IMPLEMENTATION HERE TO COUNT WORDS IN MULTPLE THREADS
  //============================================================

  auto sorted_wcounts = sort_characters_by_wordcount(wcounts);

  // results
  for (const auto& entry : sorted_wcounts) {
    std::cout << entry.first << ", " << entry.second << std::endl;
  }

}
```
The code leaves several sections blank.  It is your job to fill in these sections.  In particular, you will need to

1. Modify `count_character_words` to insert word counts into the *Character* → *Word Count* map in a thread-safe way.
2. Implement the comparator `wc_greater_than` for sorting characters by number of spoken words, descending.
3. Write multithreading code in the `main` method to parse each file and populate the word-count map.

#### Accessing the Data

The data files for this exercise need to be someplace where your program can find them.  One way is to hard-code the path in the list of filenames.  However, this will not be very portable between machines.  

The easier way is to copy the data over to the same directory as the executable (or the directory you run the executable from), then use relative path names in code to load the files.  If you are using CMake, this will automatically be taken care of for you.

A third option is to modify your project settings so that the working directory contains the `data` folder.  You can do this in Visual Studio by right-clicking on your project and selecting `Properties > Debugging > Working Directory`.  By default this is set to the project directory, so as long as the `data` folder is in the project directory itself, it can be found using relative pathnames.  Otherwise, you can set an appropriate working directory to make this easier.

#### Thread Safety

If two threads try to modify the *Character* → *Word Count* map concurrently, the behaviour is undefined.  Sometimes this will throw an error immediately, sometimes everything will *appear* to work correctly -- as in it won't crash -- but your results will seem to vary slightly every time you run your program.  If this happens, you'll often get an error when the program exits.

If the program doesn't crash right away, these concurrent modification errors can be very difficult to track down and fix.  If ever your program crashes when it closes, it's because you have overrun some memory somewhere.  It was possibly overwriting some other variables in the process, but didn't do enough damage to trigger an error at the time.  You see the error at the end as the program tries to clean up its allocated memory and notices... hey, this vector/map/array was supposed to be done by now, but seems to keep going past the end!

You need to protect access to your map if it is shared between threads by applying *mutual exclusion*.  For this, you have access to the standard library implementation `<mutex>`, which provides `std::mutex`, `std::lock_guard`, and `std::unique_lock`.

As you are protecting access to your shared map, think about critical section *localization*.  What is the smallest section of code that needs to be protected?  The narrower you can make this region, the more the multiple threads can execute in parallel.  If you lock your mutex at the beginning of a method and unlock it at the end, then calls to that method will behave as if called sequentially.  You want to try to maximize concurrency.

### Questions

- How can you test your program without needing to manually go through all the dialogue in Shakespeare's plays?
- Has writing this code multithreaded helped in any way?
- As written, if a character in one play has the same name as a character in another -- e.g. *King* -- how can you treat them as separate in the single map?

## Part 2 -- System Logger

We are going to write a small system logger example.  Most operating systems have a single system log file containing messages from many processes.  These processes need to coordinate somehow so that no process interrupts another as they are writing to the system log.  Otherwise, the log messages could be garbled.  In practice, this works by calling a kernel-specific log function that handles the synchronization internally.  However, we will take a different approach, where the processes themselves coordinate using *mutual exclusion*.  We will write two executables.  One which acts as a logging client, writing to the system log, and one parent program which will launch several logging clients.

Mutual exclusion between processes requires us to go outside standard C\+\+.  We have provided the [CPEN 333 Library](https://github.com/cpen333/library.git) to help with this.

### Accessing the Library

Within the course library exists an `include` folder.  The library is what's known as a *header only library*, which means all the code is contained within headers.  All you need to do is ensure that the `include` folder is searched when looking for the appropriate header files.

In Visual Studio, you can add a directory to the list of searched folders by adding it to the `Include Directories` variable.  This can be accomplished through the menus by going to `Project > Properties > Configuration Properties > VC++ Directories` and adding the full path name to `Include Directories` in the right-hand pane.

You should now be able to include the inter-process mutex header into your source files using
```cpp
#include <cpen333/process/mutex.h>
```

### Multiple Processes

We are going to create a small system containing two executables: a logging client, which we will call `system_logger_child`, and a parent process that will launch a set of clients called `system_logger_parent`.

#### Multiple Dependent Projects in Visual Studio

In Visual Studio, to create two output executables we need to create two projects within our solution.  You can add a new project to your existing solution by right-clicking on the solution and selecting `Add > New Project`.  Each project should contain only the source files that will be compiled for that particular executable.

By default, the executables created in Visual Studio will match the project name.  We can change the output executable name through the menus: `Project > Properties > Configuration Properties > General` and changing the `Target Name` property in the right-hand pane.  We will want our client to be called `system_logger_child`, and our driver to be called `system_logger_parent`.

The parent executable will try to run the child, which means we have a project dependency.  If we build the parent executable, we will also want to ensure our child is built.  You can add a project dependency in Visual Studio by selecting the parent project and going to `Project > Project Dependencies...`.  Select the child project from the list.

Since the parent will try to launch the child process, it also needs to know how to find it.  The easiest way to do this is to change the output directories of our projects (where the executable files go), as well as our *working directory*, which is where the executables are run from.  Let's put them all in the Solution directory in a folder named `bin`.  For each project,
- Change the binary output directory by going to `Project > Properties > Configuration Properties > General` and changing the `Output Directory` to `$(SolutionDir)\bin`.
- Similarly, for each project change the working directory by going to `Project > Properties > Configuration Properties > Debugging` and changing the `Working Directory` to `$(SolutionDir)\bin`.

The two executables should then be built into the same folder, and will be able to see each other by calling the relative executable `"./system_logger_child"` or `"./system_logger_parent"` -- note the `"./"` at the beginning which tells it to search the current directory.

By default, Visual Studio will select a single project to run when you hit the debug button.  You can change this behaviour so that it will run the current project.  Right-click on the Solution in the Solution Explorer and select `Set Startup Projects...`.  In the right-hand pane select `Current Selection`.  Alternatively, you can right-click any project and select `Debug > Start new instance` to run a particular project.

#### Multiple Dependent Projects on Other IDEs

For other IDEs and operating systems, I leave it up to you to determine how to compile multiple executables, set up dependencies, and change the output and working directories.  If using CMake to generate your projects (e.g. for Xcode), then the provided `CMakeLists.txt` is already configured to set up these properties for you.

### Writing the System Logger

Much of the code is provide for you.  The *child* process accepts three arguments from the command-line in the `argv` vector: a name, a logfile path, and a number of messages to print (**Note:** the zeroeth entry in `argv` is always the executable name).  The main method parses the inputs and runs the logger client function `run_logger`.  The logger than prints a number of messages, both the the log file and to standard output, at random time intervals.

Your job is to make the logging function run so that it is synchronized with other processes using mutual exclusion.  You will need to use the `cpen333::process::mutex` for this, and perhaps a lock.

`system_logger_child.cpp`:
```cpp
#include <fstream>
#include <iostream>
#include <string>
#include <random>
#include <thread>

#include <cpen333/process/mutex.h>

void run_logger(const std::string& processname,
                const std::string& filename,
                int message_count) {

  //=================================================
  //  IMPLEMENT THREAD/PROCESS SAFETY IN THIS METHOD
  //=================================================

  std::default_random_engine eng;
  std::uniform_int_distribution<int> rnd(20, 150);

  // mutex
  cpen333::process::mutex mutex("system_logger");

  for (int i=0; i<message_count; ++i) {
    std::this_thread::sleep_for(std::chrono::milliseconds(rnd(eng)));

    // open file for "append"
    std::ofstream logfile;
    logfile.open(filename, std::ios_base::out | std::ios_base::app);
    logfile << processname << ": " << "message " << i << std::endl;
    std::cout << processname << ": " << "message " << i << std::endl;
    logfile.close();

  }

}

int main(int argc, char* argv[]) {

  if (argc < 4) {
    std::cout << " Error: need to pass in a process name, logfile, and number of messages" << std::endl;
    return -1;
  }

  run_logger(argv[1], argv[2], std::atoi(argv[3]));

  return 0;
}
```

The parent process spins off a collection of subprocesses.  It should then wait for all of these subprocesses to finish before disposing of the interp-process mutex.

`system_logger_parent.cpp`:
```cpp
#include <cpen333/process/subprocess.h>
#include <cpen333/process/mutex.h>

int main() {

  // shared inter-process mutex
  cpen333::process::mutex mutex("system_logger");

  // list of child processes
  std::vector<cpen333::process::subprocess> processes;

  const int nprocesses = 20;
  const int nmessages = 100;
  const std::string logfile = "system_log.txt";

  for (int i=0; i<nprocesses; ++i) {
    std::vector<std::string> cmd;

    //=================================================
    //  RUN CHILD PROCESSES (system_logger_child)
    //      Process arguments:
    //          - a name for identification in the lag
    //          - a log filename to append to
    //          - a number of messages to append
    //=================================================

  }

  // wait for processes to finish
  for (auto& process : processes) {
    process.join();
  }

  // unlink mutex since we and all child processes are done with it
  mutex.unlink();

  // pause, waiting for keyboard input
  cpen333::pause();
  return 0;
}
```

Your job is to write the section that creates the subprocesses, calling `system_logger_child` and passing it the correct arguments.

**Warning:** Mac and Linux users beware!  The inter-process mutex has kernel persistence, meaning that if you accidentally leave a mutex locked when your process ends, that mutex will remain locked the next time you try to run your program.  You will need to call
```cpp
cpen333::process::mutex::unlink("system_logger");
```
to *unlink* it before your code will run again.

### Questions

1. What happens if you don't use a mutex to enforce mutual exclusion?
2. What is the smallest section of code that needs to be guarded?
