---
layout: lab
title:  Lab 1 - Introduction to C++
date:   2017-07-14 17:50:00
authors: [C. Antonio Sánchez]
categories: [labs, c++]
usemath: true

---

# Lab 1 -- Introduction to C++
{:.no_toc}

In this lab, we will get some practice with C\+\+ and object oriented programming.  Our test-case is a simplified 1D car simulator.  It is assumed you have read most of the [class notes on C\+\+](https://cpen333.github.io/lectures/cplusplus/).  To help get you started, some of the code is posted on GitHub [here (https://github.com/cpen333/lab1)](https://github.com/cpen333/lab1).

Feel free to discuss approaches and solutions with your classmates, but labs are to be completed individually.  Each student is expected to be able to answer questions about the content, describe their work, and be able to reproduce their code (or parts thereof).

{% include toc.html %}

## Part 1: Namespaces

Namespaces allow a programmer to group relevant code together and prevent *name collisions* -- two different methods with the exact same name -- by providing a *namespace scope*.  Methods and variables within the namespace can be referenced from outside by using the special namespace prefix.

### Q1:  A mini 1D physics library

We will create a small 1D physics library and simulator.  Start by creating a new header file named `physics.h`, and add the following code to it:
```cpp
#ifndef PHYSICS
#define PHYSICS

namespace physics {

   // your code will go in here

}

#endif // PHYSICS
```
The `#ifndef ... #endif` part is called an *include guard*.  This protects your code from defining the same functions or classes over and over again if this file happens to be `#include`-ed multiple times.

Add the following functions inside the namespace:

```cpp
double compute_position(double x0, double v, double dt);
double compute_velocity(double v0, double a, double dt);
double compute_velocity(double x0, double t0, double x1, double t1);
double compute_acceleration(double v0, double t0, double v1, double t1);
double compute_acceleration(double f, double m);
```

If you need to brush-up on your 1D mechanics: position, velocity, accelration and force are related through

$$v = \frac{dx}{dt} \approx \frac{x_1-x_0}{t_1-t_0}$$

$$a = \frac{dv}{dt} \approx \frac{v_1-v_0}{t_1-t_0}$$

$$f = m a$$

Note that we have a couple pairs of functions with the exact same name, but with a different number of arguments.  This is called *function overloading*, and is allowed in C\+\+ as long as either the number of arguments or the types of the arguments differ.  The compiler will figure out which function to use based on what you call it with.

**Note:** Normally, you would only *declare* the functions in the `physics.h` file and put the implementation in a separate `physics.cpp` file.  However, since these methods are quite short, you may wish to put the implementation directly in the header.  In order to prevent "multiple definition" errors, however, you will need to mark the functions as `inline` (i.e. `inline double compute_position(...)`).

### Q2: A simple car simulator

Next, create file called `car_simulator.cpp`.  We are going to use our mini physics library to simulate a car driving down a straight road.  Start by including your library, and create a `main` function.  We will also do some C\+\+ I/O using the `<iostream>` library:
```cpp
// CAR_SIMULATOR
#include <iostream>
#include "physics.h"

int main() {

  // read in car mass
  std::cout << "Enter the mass of the car (kg): ";
  double mass;
  std::cin >> mass;
  
  // read in engine force
  std::cout << "Enter the net force of the engine (N): ";
  double engine_force;
  std::cin >> engine_force;
  
  // read in drag area coefficient
  std::cout << "Enter the car's drag area (m^2): ";
  double drag_area;
  std::cin >> drag_area;
  
  // read in time step
  std::cout << "Enter the simulation time step (s): ";
  double dt;
  std::cin >> dt;
  
  // read in total number of simulation steps
  std::cout << "Enter the number of time steps (int): ";
  int N;
  std::cin >> N;
  
  // initialize the car's state
  double x = 0;  // initial position
  double v = 0;  // initial velocity
  double a = 0;  // initial acceleration
  double t = 0;  // initial time
  
  // run the simulation
  for (int i=0; i<N; ++i) {
    
    // TODO: COMPUTE UPDATED STATE HERE
    
    t += dt;  // increment time
    
    // print the time and current state
    std::cout << "t: " << t << ", a: " << a 
      << ", v: " << v << ", x: " << x << std::endl;
  }

  return 0;
}
```
There's a lot to digest in the above code if you have never used stream operators in C\+\+.  First, note the *standard namespace* `std`, where most of the C\+\+ library functions live.  We have introduced the input and output streams:

- `std::cin`: reads from standard input
- `std::cout`: writes to standard output

We also see the stream operators `<<` and `>>`.  The first *sends* data into the stream, and the second *reads* data from the stream.  The advantage of stream operators over your standard `printf`/`scanf` is that they auto-detect the variable type and format it appropriately.  In the code above, we read both doubles and an integer using the same operator.  You can also write your own versions of the operators for custom classes (we will see this in Part 2).

The `std::endl` operator adds an end-of-line character at the end of the stream, and "flushes" the output so everything gets printed to the terminal.

#### Implementing the physics

As a car picks up speed, there's a build-up of air-resistance that adds a force in the opposite direction of the car, trying to slow it down.  The drag force is computed as

$$f_d = \frac{1}{2}\rho A C_d\, v^2$$

where $$\rho$$ is the air-density, $$A$$ is the projected surface-area of the car and $$C_d$$ is the car's [*drag coefficient*](https://en.wikipedia.org/wiki/Automobile_drag_coefficient).  This drag force will counter-act the engine's drive force, causing your car to reach a top speed (or slow down if you take your foot off the gas).  A car's *drag area* is the product of the projected surface area and drag coefficient, $$A C_d$$.  Air density at sea-level is about $$\rho=1.225$$ kg/m<sup>3</sup>.  Typical drag coefficients and surface areas for a variety of vehicles can be found [here](https://en.wikipedia.org/wiki/Automobile_drag_coefficient#Drag_area) (e.g. a Toyota Prius has $$A C_d \approx 0.58$$m<sup>2</sup>).  We are going to ignore a lot of other contributing factors to a car's motion, like gear ratios and rolling friction.

To update the car's state, you will need to compute the updated force, acceleration, velocity, and position.  Use your physics library functions for this.  Remember that since your library exist in the namespace `physics`, you will need to prefix the function names with `physics::` to access them.

#### Using your simulator

Compile and run your simulator.  Try it for a variety of inputs.  An average car weighs about 1500Kg and can produce about 750 N of driving force in its highest gear.  See if you can estimate a top speed.

## Part 2: Object Oriented Programming

We are now going to modify our car simulator, encapsulating some of the functionality into various classes.

### Q1: Object-Oriented implementation of the simulator

A `State` consists of a position, velocity, acceleration, and time.  We want these attributes to be **public** so they can be accessed directly.  Will will also give the state a `set(...)` method so we can easily set all the attributes in one go.


A `Car` has a model name, mass, maximum engine force, and drag area.  The car will also have a `state` associated with it.  We'll give the car the following methods:
```cpp
class Car {
 public:
  // constructor
  Car(std::string model, double mass, double engine_force, double drag_area);
  
  std::string getModel();    // gets the model name
  double getMass();          // mass of the car
  void accelerate(bool on);  // turn the accelerator on/off
  void drive(double dt);     // drive the car for an amount of time dt
  State getState();          // returns the car's state
};

```
We can represent the two classes and the relationship between them with a *class diagram* that looks like this:

![car class diagram]({{site.url}}/assets/labs/cplusplus/car_class_diagram.png)

The car has several **private** member variables (indicated with a `-` prefix) and **public** member functions (indicated with a `+` prefix).  The state has **public** member variables and a single **public** member function.  Each car can access and modify a state, but a state cannot access or modify a car.

#### Creating the `State` class

We will start by creating the `State` class that will hold the car's position, velocity, acceleration, and time information.

1. Create a new file called `State.h`.  In this file, implement the basic class according to the class diagram in the previous section. Since this is a small class, we will put the entire implementation within the header file.  You may wish to use *include guards* to prevent the state definition from being included multiple times.
2. Create a new main program that creates an *instance* of a `State`.  Call the `set(...)` function then print out all the values of the state object to make sure it's working properly.  What happens if you print the values before calling `set(...)`?
3. Add a default constructor that initializes everything in the state to zero, and an overloaded constructor that allows you to initialize the member variables.
4. Rather than manually printing out the state information, we will overload the *stream* operator.  Below your `State` class, add the following:<br/><br/>
   ```cpp
   class State {
      ...   
   };
   
   // prints out a State class information
   inline std::ostream& operator<<(std::ostream& os, State& state) {  
      os << "t: " << state.time << ", x: " << state.position 
         << ", v: " << state.velocity << ", a: " << state.acceleration;
      return os;
   }  
  
   ```
   This allows us to use `std::cout << state << std::endl` to print out all our state's information.  We were able to access the state's member variables directly because they are declared as **public**.  If they aren't, you will have to use *accessor methods* to get the desired information (or declare the method as a `friend`, but that is beyond the scope of this course).  Since this function is defined in the header (as opposed to only being *declared*), we need to mark it as `inline`.  This only applies to stand-alone functions, not classes or member functions.
   
#### Creating the `Car` class

Next we will create the `Car` class, which will do most of the work for driving.

1. Create a new file called `Car.h`.  In this file, complete the class *declaration*: declare all the necessary public/private member variables and functions, but do not write implementations in the header.  Note that you may need more variables than what appears in the class diagram.  Since the class diagram leaves some things unspecified, it is up to you to fill in the details however you like.
2. Create a new file called `Car.cpp`.  In this file, write all the implementation details for the member functions.  If you're not sure how to do this, check the [class notes]({{site.url}}/lectures/cplusplus/#separating-classes-into-header-and-source-files).<br/>
**Notes:**
  - To use `std::string`, you will have to include the C\+\+ header `<string>`. If you've never used the `std::string` class before, it acts a bit like c-strings (`char*`), but is an object that has member functions for computing things like the length.
  - You should use the `physics.h` library you developed in Part 1 inside the `drive(...)` function implementation.
  - The car's state should be initialized to zero on construction
3. Override the *stream* operator for your `Car` class to print some of its internal details, and its current state.

#### Creating the simulator

The simulator will be our main program that will drive our cars.

1. Create a file named `drag_race.cpp`, and add a main method that looks like this:
   ```cpp
   // OBJECT-ORIENTED CAR SIMULATOR
   #include <iostream>
   #include <string>
   #include "State.h"
   #include "Car.h"
   
   int main() {
   
     Car car1("Mazda 3", 1600, 790, 0.61);
     Car car2("Toyota Prius", 1450, 740, 0.58);
   
     // drive for 60 seconds
     double dt = 0.01;
     
     // GO!!!!
     car1.accelerate(true);
     car2.accelerate(true);
     for (double t = 0; t <= 60; t += dt) {
       car1.drive(dt);
       car2.drive(dt);
       
       // TODO: print out who's in the lead
     }
   
     return 0;
   }
   ```
   Fill in the details to print out the current leader.
2. Modify the program so that the race between the two cars ends when they both cross the 402.3m mark (a quarter mile) and print out the winner.


### Q2: Car Inheritance

Let's say we wanted to create a fleet of one hundred [Evo](https://www.evo.ca) cars (they are all Toyota Prius hybrids).  We *could* use our constructor and set the car properties each time.  *OR* we can create a `Prius` class with a default constructor that automatically sets the appropriate properties.  In this section, we will create a class hierarchy for our cars and set an entire fleet loose on the highway.

1. Create a car hierarchy of classes consisting of a Prius, Mazda 3, and Tesla Model 3. Each should have only a default constructor which calls the parent `Car` constructor with some appropriate values.  Since these implementations should be quite short, you can fully implement them within the header files.  For example, a Prius implementation might look like this:
   ```cpp
   // SAMPLE PRIUS IMPLEMENTATION
   class Prius : public Car {
    public:
     // constructor
     Prius() : Car("Toyota Prius", 1450, 740, 0.58) {}
   };
   ```
2. Modify `Car.h` so that the `drive(...)` method is declared as `virtual`:
   ```cpp
   virtual void drive(double dt);
   ```
   This will allow children of the `Car` class to override `drive(...)`, using *polymorphism*.
3. Create a fourth car type, `Herbie`, that overrides the `drive(...)` function.  Herbie doesn't follow the laws of physics, so you can let it do whatever you like (e.g. ignore air resistance, or random velocities, etc...).  For Herbie to access some of the member variables in `Car` directly, you will need to change their access to **protected**:
   ```cpp
   class Car {
     protected:
      State state;
      ...
   };
   ```
<img src="{{site.url}}/assets/labs/cplusplus/car_hierarchy.png" alt="Car hierarchy" width="80%"/><br/><br/>
4. Create a new program in a file named `highway.cpp`.  In the main method, create a mixed fleet of 100 cars.  Start them all and control their speed so that they remain driving at around 100km/h (27.8m/s).  Let them drive for a few minutes (simulation time) and print out their positions.  If you can't control Herbie, that's fine, he tends to veer off the road anyways.<br/><br/>
**Notes:**
  - You can store all the cars in an array of `Car`s, or if you're adventurous, you can use the STL [vector](http://en.cppreference.com/w/cpp/container/vector) container defined in the `<vector>` header.  You can create a vector and add to it with
   ```cpp
   std::vector<Car> cars;
   cars.push_back(Prius());
   cars.push_back(Tesla3());
   cars.push_back(Herbie());
   ```
   Vectors support for-each loops, so you can easily loop through all cars with
   ```cpp
   // loop through each car by reference
   for (Car& car : cars) {
     car.drive(dt);
   }
   ```
5. Modify `Car.h` to *remove* the `virtual` declaration on `drive(...)`.  What does `Herbie` do now?  Is it what you expect?  *Should* it be what you expect?

### Q3: Pointers and Memory Management

In Q1, the car's `getState(...)` method returns a `State` object.  What happens if you modify this returned state?  Does it affect the car's internal state?  Try it.

Every time you pass around objects by value a *copy* is made.  This may be fine if the objects are small, but as they get bigger this can add a lot of overhead.  In the case of our car's state, we may also want the ability to modify the car's position or velocity directly.

- Modify your code so that rather than returning a copy of the car's state, `getState(...)` returns a *pointer* to the internal state.  You may need change other code to access position/velocity using the `->` operator.

Whenever you create an instance of a class like you would any other primitive variable type, it allocates and stores the object on the [stack](http://gribblelab.org/CBootCamp/7_Memory_Stack_vs_Heap.html).  For small objects, this is usually fine.  If, however, your objects are quite large, or you are creating a lot of them, it's much more efficient to create them on the [heap](http://gribblelab.org/CBootCamp/7_Memory_Stack_vs_Heap.html).  The heap also allows you to create dynamically sized memory blocks.

- Modify the `highway.cpp` program to instead create cars on the heap.  There are several ways you can accomplish this:
   ```cpp
   Car* cars1[100];                  // a fixed-size array of 100 car pointers
   Car** cars2 = new Car*[100];      // a dynamic array of 100 car pointers
   std::vector<Car*> cars3(100);     // a vector filled with 100 car pointers
   ```
   Note that if you create a sized vector like the third option above, do not add items using `vector.push_back(...)` since the vector already contains 100 items.  You can assign the pointers directly just as if it were an array:
   ```cpp
   for (int i=0; i<100; ++i) {
     cars1[i] = new Prius();   // fixed-size array
     cars2[i] = new Mazda3();  // dynamic array
     cars3[i] = new Tesla3();  // vector
   }
   ```
   Be sure to free all the memory when you are done using `delete` statements.  For the dynamic array, not only will you need to free each `Car` individually, but also the array itself:
   ```cpp
   for (int i=0; i<100; ++i) {
     delete cars1[i];  // fixed-size array
     delete cars2[i];  // dynamic array
     delete cars3[i];  // vector
   }
   delete[] cars2;  // free dynamic array
   ```

## Submitting Your Lab

You must demonstrate your completed lab to one of the TAs during a lab session.  He or she may ask you some questions about your work to test your understanding.  Each lab is graded out of 10, and is due by the end of the following lab period (i.e. Lab 1 is due by the end of the Lab 2 time-slot).