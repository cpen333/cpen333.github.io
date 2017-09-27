---
layout: lab
title:  Lab 4 - Shared Memory
date:   2017-09-28 12:00:00
authors: [C. Antonio Sánchez]
categories: [labs, memory]

---

# Lab 4 -- Shared Memory
{:.no_toc}

In this lab we are going to practice with sharing memory between processes.  In the first exercise, we're going to implement a maze complete with "runners" trying to navigate it. A separate user interface process will access the maze to show us how everyone is doing.

In the second exercise, we're going to use shared memory to monitor whether one of our processes is still running, and if it isn't we will re-launch it.  This is a common approach in malware to prevent users from shutting it down completely.

To help get you started, some of the code is posted on GitHub [here (https://github.com/cpen333/lab4)](https://github.com/cpen333/lab4).

Feel free to discuss approaches and solutions with your classmates, but labs are to be completed individually.  Each student is expected to be able to answer questions about the content, describe their work, and reproduce their code (or parts thereof).

{% include toc.html %}

## Part 1 -- The Maze Runner

In the `data/` folder on [GitHub](https://github.com/cpen333/lab4/tree/master/data) you will find a few maze layouts, with `'X'` indicating a wall, `'E'` indicating the exit, and `' '` indicating an empty space.  We are going to create a small system of processes to run the maze challenge.  It will consist of three executables:
- `maze_runner_main`: loads the maze file and initializes the shared memory
- `maze_runner_ui`: shows a display of the maze and any runners in it
- `maze_runner`: a participant who enters the maze and tries to find his/her way out

### Shared Memory

To implement interprocess shared memory, we will use the CPEN 333 [course library](https://github.com/cpen333/library).  Make sure the `include/` folder is somewhere on the Include Path (or Header Search Path), so your compiler/IDE can find it.

The data structure to be shared, `SharedData` is broken down as follows:
```cpp
#define MAX_MAZE_SIZE 80
#define MAX_RUNNERS   50

struct MazeInfo {
  int rows;           // rows in maze
  int cols;           // columns in maze
  char maze[MAX_MAZE_SIZE][MAX_MAZE_SIZE];  // maze storage
};

struct RunnerInfo {
  int nrunners;               // number runners
  int rloc[MAX_RUNNERS][2];   // runner locations [col][row]
};

struct SharedData {
  MazeInfo minfo;    // maze info
  RunnerInfo rinfo;  // runner info
  bool quit;         // tell everyone to quit
};
```
The memory layout of the `SharedData` structure, if expanded, would look something like this:
```
  Offset     Variable
==========================
[ 0x0000 ]  minfo.rows
[ 0x0004 ]  minfo.cols
[ 0x0008 ]  minfo.maze[][]
[ 0x1908 ]  rinfo.nrunners
[ 0x190C ]  rinfo.rloc[][]
[ 0x1A9C ]  quit

size: 0x1AA0 = 6816 bytes
```

**Aside:** with many compilers an `int` is 4 bytes, but *this is not guaranteed*.  In fact, this leads to a gotcha: if two programs are compiled with different compilers, the byte-size of `int` may differ and the programs will no longer agree on the memory layout above.  Compilers may also add extra space between items to satisfy memory alignment conditions, such as having 64-bit integers aligned on 64-bit boundaries.  All this to say that if sharing memory between processes, the programs should be compiled with the same compiler.

Remember that when sharing memory between processes, we cannot store pointers since they would point to somewhere outside of the shared memory block.  This is why the above structs are of known fixed sizes.

Rather than working with memory addresses and casting to our desired types, we will use the [`shared_object<...>`](https://cpen333.github.io/library/docs/html/classcpen333_1_1process_1_1shared__object.html) class from the CPEN 333 library.  This uses template programming to handle all the type casts for us.  For example, we can create an instance of shared memory that stores an object of `SharedData` using
```cpp
#include <cpen333/process/shared_memory.h>
//...
cpen333::process::shared_object<SharedData> memory("lab4_maze_runner");
```
We can then use `memory` as if it were a pointer to the shared `SharedData` structure:
```cpp
int rows = memory->minfo.rows;
```

### The Main Maze Process

The purpose of the main maze process, `maze_runner_main`, is to load a maze from a file, initialize the shared memory object, and generate random starting positions for all maze runners who may join.

In the provided code, two helper functions are already implemented:
```cpp
/**
 * Reads a maze from a filename and populates the maze
 * @param filename file to load maze from
 * @param minfo maze info to populate
 */
void load_maze(const std::string& filename, MazeInfo& minfo);

/**
 * Randomly places all possible maze runners on an empty
 * square in the maze
 * @param minfo maze input
 * @param rinfo runner info to populate
 */
void init_runners(const MazeInfo& minfo, RunnerInfo& rinfo);
```

Your task is to create the shared memory object and initialize *all* members.  The main method currently looks as follows:
```cpp
int main(int argc, char* argv[]) {

  // read maze from command-line, default to maze0
  std::string maze = "data/maze0.txt";
  if (argc > 1) {
    maze = argv[1];
  }

  //===============================================================
  //  TODO:  CREATE SHARED MEMORY AND INITIALIZE IT
  //===============================================================

  std::cout << "Keep this running until you are done with the program." << std::endl << std::endl;
  std::cout << "Press ENTER to quit." << std::endl;
  std::cin.get();

  //===============================================================
  //  TODO:  INFORM OTHER PROCESSES TO QUIT
  //===============================================================
  return 0;
}
```
When the user hits the enter key, your code should use the shared memory to tell all other processes to quit.

### The User Interface

The UI for this program, `maze_runner_ui`, is completely passive.  It simply displays the maze and the locations of any `runners` in it.  It uses the [`console`](https://cpen333.github.io/library/docs/html/classcpen333_1_1console.html) class for manipulating printed locations on the console (or terminal).  Note that this class uses special ANSI codes for the manipulation, so you need it a full-fledged console or terminal application that supports these for the display to work properly.  Windows `cmd.exe` -- launched by Visual Studio by default when running your program -- will do the job.  In Xcode, you will need to edit your [runtime *scheme*](https://stackoverflow.com/a/40361384/4266470) to use the Terminal app.

Updating the display is handled by the `MazeUI` class, for which the bulk of the code is provided.

```cpp
/**
 * Handles all drawing/memory synchronization for the
 * User Interface process
 * ====================================================
 *  TODO: ADD ANY NECESSARY MUTUAL EXCLUSION
 * ====================================================
 *
 */
class MazeUI {
  // display offset for better visibility
  static const int XOFF = 2;
  static const int YOFF = 1;

  cpen333::console display_;
  cpen333::process::shared_object<SharedData> memory_;
  cpen333::process::mutex mutex_;

  // previous positions of runners
  int lastpos_[MAX_RUNNERS][2];
  int exit_[2];   // exit location

 public:

  MazeUI() : display_(), memory_(MAZE_MEMORY_NAME), mutex_(MAZE_MUTEX_NAME){

    // clear display and hide cursor
    display_.clear_all();
    display_.set_cursor_visible(false);

    // initialize last known runner positions
    for (size_t i=0; i<MAX_RUNNERS; ++i) {
      lastpos_[i][COL_IDX] = -1;
      lastpos_[i][ROW_IDX] = -1;
    }

    // initialize exit location
    exit_[COL_IDX] = -1;
    exit_[ROW_IDX] = -1;

    //===========================================================
    // TODO: SEARCH MAZE FOR EXIT LOCATION
    //===========================================================

  }

  /**
   * Draws the maze itself
   */
  void draw_maze() {
    static const char WALL = 219;  // WALL character
    static const char EXIT = 176;  // EXIT character

    MazeInfo& minfo = memory_->minfo;
    RunnerInfo& rinfo = memory_->rinfo;

    // clear display
    display_.clear_display();

    // draw maze
    for (int r = 0; r < minfo.rows; ++r) {
      display_.set_cursor_position(YOFF+r, XOFF);
      for (int c = 0; c < minfo.cols; ++c) {
        char ch = minfo.maze[c][r];
        if (ch == WALL_CHAR) {
          std::printf("%c", WALL);
        } else if (ch == EXIT_CHAR){
          std::printf("%c", EXIT);
        } else {
          std::printf("%c", EMPTY_CHAR);
        }
      }
    }
  }

  /**
   * Draws all runners in the maze
   */
  void draw_runners() {

    RunnerInfo& rinfo = memory_->rinfo;

    // draw all runner locations
    for (size_t i=0; i<rinfo.nrunners; ++i) {
      char me = 'A'+i;
      int newr = rinfo.rloc[i][ROW_IDX];
      int newc = rinfo.rloc[i][COL_IDX];

      // if not already at the exit...
      if (newc != exit_[COL_IDX] || newr != exit_[ROW_IDX]) {
        if (newc != lastpos_[i][COL_IDX]
            || newr != lastpos_[i][ROW_IDX]) {

          // zero out last spot and update known location
          display_.set_cursor_position(YOFF+lastpos_[i][ROW_IDX], XOFF+lastpos_[i][COL_IDX]);
          std::printf("%c", EMPTY_CHAR);
          lastpos_[i][COL_IDX] = newc;
          lastpos_[i][ROW_IDX] = newr;
        }

        // print runner at new location
        display_.set_cursor_position(YOFF+newr, XOFF+newc);
        std::printf("%c", me);
      } else {

        // erase old position if now finished
        if (lastpos_[i][COL_IDX] != exit_[COL_IDX] || lastpos_[i][ROW_IDX] != exit_[ROW_IDX]) {
          display_.set_cursor_position(YOFF+lastpos_[i][ROW_IDX], XOFF+lastpos_[i][COL_IDX]);
          std::printf("%c", EMPTY_CHAR);
          lastpos_[i][COL_IDX] = newc;
          lastpos_[i][ROW_IDX] = newr;

          // display a completion message
          display_.set_cursor_position(YOFF, XOFF+memory_->minfo.cols+2);
          std::printf("runner %c escaped!!", me);
        }
      }
    }
  }

  /**
   * Checks if we are supposed to quit
   * @return true if memory tells us to quit
   */
  bool quit() {
    // check if we need to quit
    return memory_->quit;
  }

  ~MazeUI(){
    // reset console settings
    display_.clear_all();
    display_.reset();
  }
};
```
You have two tasks to complete:
- finish the constructor, searching for and storing the location of the maze exit
- add any mutual exclusion required to protect critical sections of code

### Maze Runner

The `maze_runner` process represents one of our maze participants.  Participants can join at any time once the `maze_runner_main` process has started.  

All the work is performed in the `MazeRunner` class.  In the constructor, the runner automatically grabs the next available runner's index from the shared memory, and identifies its starting position.  It also makes a duplicate copy of the maze layout information for its own personal use.

```cpp
class MazeRunner {

  cpen333::process::shared_object<SharedData> memory_;
  cpen333::process::mutex mutex_;

  // local copy of maze
  MazeInfo minfo_;

  // runner info
  size_t idx_;   // runner index
  int loc_[2];   // current location

 public:

  MazeRunner() : memory_(MAZE_MEMORY_NAME), mutex_(MAZE_MUTEX_NAME),
                 minfo_(), idx_(0), loc_() {

    // copy maze contents
    minfo_ = memory_->minfo;

    {
      // protect access of number of runners
      std::lock_guard<decltype(mutex_)> lock(mutex_);
      idx_ = memory_->rinfo.nrunners;
      memory_->rinfo.nrunners++;
    }

    // get current location
    loc_[COL_IDX] = memory_->rinfo.rloc[idx_][COL_IDX];
    loc_[ROW_IDX] = memory_->rinfo.rloc[idx_][ROW_IDX];

  }

  /**
   * Solves the maze, taking time between each step so we can actually see progress in the UI
   * @return 1 for success, 0 for failure, -1 to quit
   */
  int go() {
    // current location
    int c = loc_[COL_IDX];
    int r = loc_[ROW_IDX];

    std::this_thread::sleep_for(std::chrono::milliseconds(100));

    //==========================================================
    // TODO: NAVIGATE MAZE
    //==========================================================

    std::this_thread::sleep_for(std::chrono::milliseconds(100));

    // failed to find exit
    return 0;
  }

};
```
Your task is to fill in the `go()` implementation.  Your runner should navigate the maze one step at a time, update it's current location in the shared memory object,
```cpp
memory->rinfo.loc[idx_][COL_IDX] = new_col;
memory->rinfo.loc[idx_][ROW_IDX] = new_row;
```
and continue either until it reaches the exit location, or the process is told to quit.  If told to quit by use of the shared memory variable, your `go()` method should quit immediately, returning -1.

### Running the Maze

Rather than have one parent process spawn off all the child processes, we are going to run these manually.

Start by running `maze_runner_main`.  Leave the window open so you can quit all the processes later by pressing the ENTER key.

Next, run `maze_runner_ui`.  In Visual Studio, you can right-click on the project and select `Debug > Start new instance`.  In Xcode, you can select the scheme at the top and press play.  The UI program should show the maze layout with no participants.

Finally, we are going to start several runners.  Initiate the runner process several times in a row.  Each one should populate a new character in the maze.  They will each run independently, at their own rate.  New runners can be added at any time as long as the main program is still running.

You should be able to close down the UI and re-open it again without any issues.  Once opened, the new UI should display the current state of the maze and participants.

If you hit ENTER in the `maze_runner_main` process window, it should set the shared `quit` variable to `true`, and all other processes should quit almost immediately, including the runners.

### Detecting Initialization

What happens if you start the `maze_runner_ui` before you start `maze_runner_main`?  How about one of the runners?

We don't want any processes to be running before our shared memory is initialized, but how do we detect this?  If we have two processes trying to create or attach to a shared memory object, how can we tell who arrived first?

The most common way is to use something called a *magic number*.  The idea is to fill part of the memory with a special identifier that is unlikely to be mistaken for anything else.  If the magic number is present, the memory has already been initialized.  If not, the memory is uninitialized.

Add a new variable to the `SharedData` structure:
```cpp
struct SharedData {
  MazeInfo minfo;    // maze info
  RunnerInfo rinfo;  // runner info
  bool quit;         // tell everyone to quit
  int  magic;        // magic number for detecting initialization
};
```
Also come up with a fixed value you will use as your identifier.  It should be unique enough that it won't accidentally be detected in a random block of uninitialized memory.

In the `maze_runner_main` process, after the shared memory block has been initialized, set the magic number variable to your unique value.  At the end of the `main` function, clear the magic number variable -- this is to prevent any new processes from using the memory after the main program has closed, just in case the shared memory persists (e.g. on a Mac or Linux)

In both `maze_runner_ui` and `maze_runner`, before we try to use any of the shared memory, first check that it has been initialized.  If it hasn't, then print an error message and quit immediately.

### Questions:

1. In the `MazeUI` class, what is the smallest section of code that needs to be protected by mutual exclusion?  Think about what might be considered *static* memory (i.e. remains constant) and what is *dynamic* memory (i.e. changing).
2. In the `MazeRunner` constructor, why is only the access to `memory_->info.nrunners` protected?  Why can we get away with copying the runner's current location without needing to worry about mutual exclusion?
3. Did you protect access when setting and checking if the magic initialization number had been set?  

   There is actually a very subtle reason why you  *do* need to.  Because of memory caching, the order that some variables are assigned in one thread are not necessarily seen as changing in the same order in another thread.  It is possible that the magic number is viewed as being set before the rest of the shared memory block is. That is unless you use a mutex to ensure that the entire critical section is complete before someone else tries to read from it.  A mutex will enforce the reading thread to update its cache of the entire protected section.

## Part 2 -- Malware

Malware, or malicious software, can be difficult to close down completely.  If you manage to kill one process manually, you'll often find that it just starts back up again on its own.  It can feel like you're playing a game of whack-a-mole.  There are a variety of techniques that can be used to accomplish this.  We're going to explore one: have two processes running, and if either detects that the other one has been closed, it starts another instance of the process back up again.

How can we detect if another process is still running?  At first you might think we could use the RAII pattern: as a process closes down, notify the other process so it knows to start it back up immediately.  Unfortunately, when a process is forcibly closed, no destructors are called (i.e. there is no *stack-unwinding*).  Instead, the entire memory block is reclaimed immediately by the operating system.

What other trick can we use?  

Did you have, or know someone who had, a protective parent?  
> *I want you to check in every hour so I know you're okay.*

We can use the same idea in software.  Each process is assigned a *check-in* variable, and needs to update it periodically to show that the process is still alive.  The other process can monitor this check-in variable, and if it hasn't been updated in a while, it will know something is wrong.

The following is a skeleton of the code for our malware application:
```cpp
#include <cpen333/process/subprocess.h>
#include <cpen333/process/shared_memory.h>
#include <chrono>
#include <thread>
#include <string>
#include <iostream>

// Usage:
//    malware <name> <index>
// name is any name
// index is 0 or 1
// defaults to name:malware, index:0
int main(int argc, char* argv[]) {

  // extract name and index
  int index = 0;
  std::string name = "malware";
  if (argc > 1) {
    name = argv[1];
  }
  if (argc > 2) {
    index = std::atoi(argv[2]);
  }

  std::cout << name << " " << std::to_string(index)  << " started" << std::endl;

  //========================================================
  // TODO: CREATE AND INITIALIZE SHARED MEMORY
  //========================================================

  int oindex = (index+1)%2;  // index of other malware process

  while(true) {
    std::cout << name << " " << std::to_string(index)  << " running" << std::endl;

    //=======================================================
    // TODO: CHECK IF OTHER PROCESS MISSED CHECK-IN(S)
    //       - LAUNCH IF NOT RESPONDING
    //=======================================================

    std::this_thread::sleep_for(std::chrono::seconds(10));
  }

  return 0;
}
```
The program extracts a name and index number (0 or 1) from the command-line arguments.  It then continuously loops, printing its name and index and waiting for the next check-in.

**Your task:** as long as one process is still running, make sure the other process doesn't remain closed for more than 10 seconds.

To do this you can implement a version of the check-in system using shared memory.  It is up to you what to store.  You could use counters that need to be incremented, or explicitly store times of last check-in.  You may or may not need a way to initialize the memory as well.

**Notes:**
- Be careful not to continuously launch processes faster than you can close them manually!!!  Unlike threads, creating too many process will not throw a *system error*.  Instead, they will simply bog down your computer until it becomes unusable (at which point you've actually created malware).
- When launching a new process, you'll want to create it in *detached* mode.  Otherwise when the parent process is closed, the child will also be closed automatically.  This can be done by looking at the constructor for [`subprocess`](https://cpen333.github.io/library/docs/html/classcpen333_1_1process_1_1subprocess.html).
- Sometimes you might just *barely* miss a check-in due to time slicing and race conditions.  You'll want to be sure that the other process is still down after a reasonable amount of time, but still not let it remain closed for more than 10 seconds.  You may need to change any sleep timings.

### Questions
1. How did you implement your shared memory?
2. Did you protect access to that shared memory using mutual exclusion?  Did you *need* to? (the answer depends on exactly how you implemented the check-in process)
