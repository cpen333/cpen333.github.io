---
layout: lecture
title:  Introduction to C++
date:   2017-07-12 17:50:00
authors: [Paul Davies, C. Antonio Sánchez]
categories: [lectures, c++, object oriented]

---

# Introduction to C\+\+ 
{:.no_toc}

**Authors:** Paul Davies (UBC), C. Antonio Sánchez (UBC)

{% include toc.html %}

## Learning Goals

In this lecture, we cover some of the history and basics of C\+\+.  By the end, you should be able to

* create and use *namespaces* to group together related functions
* define a custom *class*, both *inline* and in separate source and header files
* distinguish between *public* and *private* member variables and functions
* create a set of overloaded class *constructors* for your custom class
* describe the difference between *references* and *pointers*
* allocate and free memory using the `new` and `delete` operators
* create a class hierarchy using *inheritance* to re-use code
* override a member function using *polymorphism*

Don't get *too* hung up on some of the details below if it sounds a little complicated.  This whole document goes through quite a bit of background to C\+\+.   We won't emphasize half as much of this in CPEN 333 as you might think, but it is useful to know and try out.  Please try to read through all the material, but don’t agonise over it if you don't get it 1st, 2nd or even 3rd time through.  As with everything, these concepts will become more natural to you with experience, so... practice, practice, practice.

## What's wrong with just 'C'?

'C' is still one of the most common languages [in use around the world](https://www.tiobe.com/tiobe-index/), particularly in Engineering and embedded systems.  However, this is slowly changing as the historical benefits are dwindling.  C\+\+ offers many advantages that make writing and managing easier:
- object-oriented design, inheritance, polymorphism, encapsulation
- a very rich function and object library
- function overloading and exception handling
- generic programming in the form of templates
- ...

We will cover some of these features in this lecture and course.  To learn about others, check out some of the links in the [Additional Resources](#additional-resources) section.

In C, programs are made-up of a set of variables and global functions.  The variables hold all the data, and the functions manipulate it.  This is easy to manage when programs and teams are relatively small.  But as these programs and systems grow in size, with larger development teams, it becomes difficult to maintain.  Due to the 'global' nature of all the variables and functions, they are visible to everyone, e.g.
```c
// global variables
int x, y, z;        // Paul's x, y, z
int x2, y2, z2;     // Antonio's x, y, z
int x3, y3;         // Sylvia's x, y
double my_speed, your_speed, their_speed;

// global functions
void update_position_xyz(int& x, int& y, int& z, double speed, double t);
void update_position_xy(int& x, int& y, double speed, double t);
double get_my_speed(void);
void set_my_speed(double speed);
double get_your_speed(void);
void set_your_speed(double speed);
...
```

Each of the function and variable names have to be *unique* to prevent name conflicts, even if they play similar roles.  Imagine working on a team of 10, 50, 100 developers and needing to continuously invent new names.  It also becomes difficult to keep track of which functions are related to (i.e. need access to) which variables.  Everything ends up in one giant melting pot of global names.

These problems only get worse as programs and teams get larger.  We need a way to manage the complexity and keep track of which code uses which variables so we can make changes more easily if he need arises.

## Managing complexity

### Static Declarations
In C, the complexity was somewhat managed using separate source files.  Each programmer would isolate their own variables and functions in a file, and could choose to hide parts from others (i.e. making them **private**) by prefixing the declarations with the word `static`.  This meant that only the functions written within *that* source file could access the `static` elements.  Anything not declared as `static` could be accessed by the whole program (i.e. were **public**).  For example,
```c
/*************************************************************
*  PaulsFile.c
**************************************************************/
// variables
static double  my_speed;      // PRIVATE to this source file

// functions
double get_my_speed(void);            // public to the world
void   set_my_speed(double speed);    // public to the world
static void my_private_fn1(void);     // PRIVATE to this file
```

```c
/**************************************************************
*  AntoniosFile.c
***************************************************************/
// variables
static double  your_speed;      // PRIVATE to this source file

// functions
double get_your_speed(void);            // public to the world
void   set_your_speed(double speed);    // public to the world
static void my_private_fn1(void);       // PRIVATE to this file
```

Programmers could now add variables and functions to their *own* source files without creating name conflicts with others.  For example, two source files could each contain a private (`static`) function called `my_private_fn1()` without causing any issues.  The two function definitions would be completely unrelated, with each used only in the context of its own file.  This allowed teams to write and manage their code, keeping related functions and variables in separate source files and only exposing the functionality needed by others.

Hiding variables and functions away from other programmers that didn't need to know the inner details also made it easier to change code, since the effects of any change were limited to that source file.  We call this concept *encapsulation*.

The `static` keyword continues to exist in C\+\+, and the practice of separating groups of related functions and variables into separate source files is still an important part of modern programming dogma.  But while keeping private function and variable names local to their containing file, what can we do with names we actually *want* to be exposed?  Imagine working on a large project with thousands of source files and millions of lines of code.  How can we prevent public name collisions on *THAT* scale?

### Namespaces

Namespaces, introduced in C\+\+, allow us to group methods and variables that originally have a *global scope* (i.e. accessible by everyone) into something with a narrower scope, called a *namespace scope*.  Inside the namespace, the methods/variables are accessed as normal.  Outside the namespace scope, we can still access those methods and variables using a special namespace prefix `my_namespace::`.

For example the following creates a namespace called `cpen333`, with a few variables and a function
```cpp
// my new namespace
namespace cpen333 {

int c = 10, d = 20;

int max(int a, int b) {
  return (a > b) ? a : b;
}

} // cpen333
```

Inside the namespace, the variables `c` and `d` can be accessed normally.  Outside the namespace, however, they must be qualiified with the prefix `cpen333::`.  For example,
```cpp
int c = cpen333::max( cpen333::c, cpen333::d );
```
can be called from the outside world.  The double-colon `::` is known as the *scope resolution operator*.  It indicates that we are accessing elements within the givennamespace scope.  We will see this scope resolution operator again later when we introduce classs.

Namespaces have become an intergral part of modern C\+\+ programming.  It is often adviced that if you are writing any code you wish to share with others, it should be contained within a custom namespace with a meaningful name (describing the purpose of your code or library).  If you have tried any C\+\+ programming or examples before, you will likely have already encountered the namespace `std`, which is the *standard* namespace.

Namespaces give us further control for grouping sets of related variables and functions together, and helps further prevent name collisions on large projects.  They can also be nested to hierarchically organize your code, each *encapsulating* a subgroup of related functions (i.e. `cpen333::math::trig::...`).

**Object Oriented Programming** (OOP) takes these *private* and *encapsulation* concepts one step further.

### Exercises

1. Create a small integer-based math library inside a custom *namespace*
  - support basic operations
  ```cpp
  int add(int x, int y);    // result = x+y
  int mul(int x, int y);    // result = x*y
  int div(int x, int y);    // result = x/y
  int pow(int x, int n);    // result = x^n
  ```
  - support several "root" functions: 
  ```cpp
  int isqrt(int x)          // integer square root
  int icbrt(int x)          // integer cubed root
  int iroot(int x, int n);  // integer n-th root
  ```
  that returns the largest integer that is less-than or equal-to the desired root (Hint: you can often use binary search for these types of problems, or logarithms if you're careful with rounding).
2. Create a `main` program that prompts the user for a value `x` and root-parameter `n`, and uses your mini library to compute and print out the *n*-th root of *x*.  Prove to the user that your answer is correct (i.e. show that `pow(r, n) <= x` and `pow(r+1, n) > x`).

## Classes

*Classes* are a great way of grouping together related variables and functions into one place.  They are devined using the `class` keyword, with the following syntax:
```cpp
class name {
 private:
  // some private variables and functions
  
 public:
  // some public variables and functions
};
```
Here `name` is the name of the class.  The body can contain `private` and `public` *members*, which can be either variables or functions.

- `private` members are accessible only to functions that are also members of the same class
- `public` members are accessible by everyone, including users of the class

By default, all members of a class are `private` unless specified otherwise.  For example,
```cpp
class Rectangle {
  int width, height;
  
 public:
  void setValues(int x, int y);
  int  getArea();
};
```
declares a class (i.e. a new *data type*, like `int` or `float`) called `Rectangle`.  This class contains four members:

- two variables of type `int` with *private* access (the default): `width` and  `height`
- and two member functions with *public* access: `setValues(...)` and `getArea()`

Because `width` and `height` are `private`, only the member functions `setValues(...)` and `getArea()` are allowed to access them.  Nobody else can even see that the variables exist.  The member functions are `public`, so anybody can call/use them to indirectly get at and manipulate the width and height.

### Creating an Object from a class

An **object** is an *instance* of a class.  By analogy with traditional variables, a `class` introduces a new *data type*, which an **abject** would be the *variable* of a given data type.  For example we might declare instances of an integer variable
```cpp
int a1, a2, a3;
```
Now that we have classes, we can write something similar to create objects:
```cpp
Rectangle r1, r2, r3;
```
Here `r1`, `r2`, and `r3` are three *instances* of a rectangle (i.e. they are rectangle "objects", or variables of type `Rectancle`).  Each rectangle has hidden within it its *own* unique member variables `width` and `height`.  In C\+\+ we say that the class `Rectangle` *encapsulates* (or forms a "wrapper" around) its member variables and functions.

### Manipulating objects

Once we have a rectangle object, we can use any of the `public` members of that objected as if they were normal functions or variables by using the dot operator `.` between the *variable name* and *member name*.  This follows the same syntax as accessing the members of a `struct` in C.  For example we can set the rectangle sizes using
```cpp
Rectangle r1, r2, r3;
r1.setValues(3, 4);
r2.setValues(5, 6);
r3.setValues(7, 8);
```

Here is a complete example:
```cpp
/**************************************************************************
* Example use of a rectangle
**************************************************************************/
#include <cstdio>   // c++ version of old-school i/o prefixed with std::

// Rectangle class
class Rectangle {
  int width, height;
  
 public:
  void setValues(int x, int y) {
    width = x;
    height = y;
  }
  
  int getArea() {
    return width*height;
  }
};

int main() {
  Rectangle r1, r2;             // two rectangle instances
  r1.setValues(3, 4);
  r2.setValues(5, 6);
  
  std::printf("r1's area is %d\n", r1.getArea());  // prints 12
  std::printf("r2's area is %d\n", r2.getArea());  // prints 30
  
  return 0;
}
```
Notices that the calls to `r1.getArea()` and `r2.getArea()` do not print the same results.  This is because each of `r1` and `r2` have theirh *own* variables `width` and `height`.  This is an **extremely important**  concept.  The functions `setValues(...)` and `getArea()` are manipulating the variables of the *object* that *invoked* them.

In the example above, we embedded the actual code for the functions `setValues(...)` and `getArea()` *inside* the class definition itself.  This is acceptable here because that code is small and simple.  However, when the functions become large and if there are many of them, it can become unwieldy to do it this way.  In general, it is good practice to separate out the declaration and implementation like this:
```cpp
/**************************************************************************
* Example use of a rectangle with separate declaration/implementation
**************************************************************************/
#include <cstdio>   // c++ version of old-school i/o prefixed with std::

// Rectangle class
class Rectangle {
  int width, height;
  
 public:
  void setValues(int x, int y);  // DECLARATION of the class function
  int getArea();                 // DECLARATION of the class function
};

// DEFINITION of the class function
void Rectangle::setValues(int x, int y) {
  width = x;
  height = y;
}

// DEFINITION of the class function
int Rectangle::getArea() {
  return width*height;
}

int main() {
  Rectangle r1, r2;             // two rectangle instances
  r1.setValues(3, 4);
  r2.setValues(5, 6);
  
  std::printf("r1's area is %d\n", r1.getArea());  // prints 12
  std::printf("r2's area is %d\n", r2.getArea());  // prints 30
  
  return 0;
}
```
This example demonstrates the *scope resolution operator* `::` for defining the class methods.  It is used to say that the functions `setValues(...)` and `getArea()` belong to the class `Rectangle`, and have access to any of its public or private members.

Apart from code management, there is little difference between defining a member function *within* a class versus outside the class.  Traditionally, definitions inside the class were considered *inline* (i.e. would duplicate code whenever the method was called, leading to faster run-times but perhaps larger binaries).  However, modern compilers will decide whether or not there are any benefits to inline-ing each member, and will optimize accordingly.  Functionally, the two definitions are the same.

### Separating classes into header and source files

Taking this one step further, C\+\+ wisdom dictates that the parts of teh class be separated into a **Rectangle.cpp** *source* file (note the **.cpp** suffix) and a **Rectangle.h** *header* file.  This is not essential, but is demonstrated below.

```cpp
/***************************************
*  Rectangle.h
***************************************/
class Rectangle {
  int width, height;
 public:
  void setValues(int x, int y);
  int getArea();
}
```

```cpp
/***************************************
*  Rectangle.cpp
***************************************/
void Rectangle::setValues(int x, int y) {
  width = x;
  height = y;
}

int Rectangle::getArea() {
  return width*height;
}
```

To use the `Rectangle` class, your main code must *include* the header file using `#include "Rectangle.h"` (note the quotes instead of `<...>`).

**ASIDE:** the quotes in an `#include "..."` directive tell the compiler to first look for the header in a directory relative to the current source file.  If the header is not found, it will search for the header in a set of system-dependent directories.  Using the `#include <...>` directive tells the compiler to go straight to the system-dependent directories.  You generally use quotes if *you* created the header and know where it is.

Note that you cannot access the class's members directly since by default they have *private* access.  If you tried, you would get a compile error:
```cpp
r1.width = 10;      // compile ERROR
int h = r1.height;  // compile ERROR
```
In an example as simple as this, it's hard to see how restricting access to the member variables may be useful.  But in bigger projects, it is important from a maintenance point of view: you are limiting access so that changes to code have limited and isolated effects.  This would allow the programmer to change how the class works internally without affecting any code that uses it.  This is the main advantage of *encapsulation*.

Ideally, objects present their external *interfaces* to the outside world (i.e. their *public* member functions) that will allow others to *get* and *set* the member variables in a way that is appropriate.  For example, you may wish to restrict rectangle widths and heights to be positive numbers.  We also don't want users of an object to write code that is dependent on knowing the internal details of an object.  For example, just like driving a car, you use the interfaces provided by the car designer (e.g. brakes, accelerator, steering wheel, etc.) and manipulate the car via those interfaces.  You don't need to know how the car actually *works* to drive; that is hidden and only of concer to the designer, not the user.

Furthermore, if all cars provide the same interfaces (as they generally do), then one car should be interchangeable with another from the user's perspective.  The car designer is free to change things like the type of engine, without affecting us users who only interact with the car through the interfaces, not the implementation.

### Constructors

What would happen in the previous example if we called the member function `getArea()` before having called `setValues(...)`?  Try it.

We get an undetermined result since the object variables `width` and `height` would not have been assigned an initial value. As programmers, it would be our responsibility to make sure we called the function `setValues()` to initialize our rectangle as soon as possible after we create an instance of it. If we forget, then we are setting ourselves up for problems later.

In order to avoid that, a class can include a special function called a *constructor*, which is automatically called whenever a new object/instance of that class is created. The constructor can perform any actions you like e.g. initialize variables, open files, turn on LEDs in a small embedded system etc., but typically it is used to initialize the object’s member variables. In a way, our `setValues(...)` function is already carrying out this task, but it is not called automatically, we have to remember to call it ourselves.

A proper C\+\+ constructor function is declared just like a regular member function, but with a name that is **identical** to the class name (that’s how the C\+\+ compiler knows which of our many functions is the constructor – it has the same name as the class). Constructor functions *do not have a return type*, not even `void`.

The Rectangle class above can easily be improved by adding a proper constructor to replace the `setValues(...)` functions,
```cpp
/**************************************************************************
* Example use of a rectangle
**************************************************************************/
#include <cstdio>   // c++ version of old-school i/o prefixed with std::

// Rectangle class
class Rectangle {
  int width, height;
  
 public:
  Rectangle(int x, int y) {  // CONSTRUCTOR
    width = x;
    height = y;
  }
  
  int getArea() {
    return width*height;
  }
};

int main() {
  Rectangle r1(3, 4), r2(5, 6);   // two rectangle instances, initialized
  std::printf("r1's area is %d\n", r1.getArea());  // prints 12
  std::printf("r2's area is %d\n", r2.getArea());  // prints 30
  
  return 0;
}
```
Notice how these arguments are passed to the constructor at the moment at which the objects of this class are created e.g. `Rectangle r1(3,4)`. The '3' becomes the argument `width` and '4' the argument `height` in the constructor function.

Now, class `Rectangle` has no member function `setValues(...)`, but instead has a constructor that performs a similar action. It initializes the values of width and height with the arguments passed to it.

**Summary:** constructors are executed automatically when a new object/instance of that class is created. They should be declared with `public` access (see above and verify this) to allow them to be called. Constructors *never* return values, they simply initialize the object.

#### Overloading Constructors

Like any other C\+\+ function (and unlike functions in simple C), a constructor can also be *overloaded*. This means that you can have several functions in your class with the same name as long as they have different numbers of arguments (or the same number but different types of arguments). The compiler will automatically call the correct constructor to match the arguments used when the object is created. For example:
```cpp
/**************************************************************************
* Example use of a rectangle
**************************************************************************/
#include <cstdio>   // c++ version of old-school i/o prefixed with std::

// Rectangle class
class Rectangle {
  int width, height;
  
 public:
  Rectangle(int x, int y) {  // CONSTRUCTOR with width/height
    width = x;
    height = y;
  }
  
  Rectangle(int x) {         // CONSTRUCTOR creating a square
    width = x;
    height = x;
  }
  
  Rectangle() {              // DEFAULT CONSTRUCTOR
    width = 0;
    heigh = 0;
  }
  
  int getArea() {
    return width*height;
  }
};

int main() {
  Rectangle r1(3, 4), r2(5);   // two rectangle instances, initialized
  Rectangle r3;                // calls default constructor
  std::printf("r1's area is %d\n", r1.getArea());  // prints 12
  std::printf("r2's area is %d\n", r2.getArea());  // prints 25
  std::printf("r3's area is %d\n", r3.getArea());  // prints 0
  
  return 0;
}
```
In the above example we also introduced a special kind of constructor: the *default constructor*. The default constructor is the constructor that takes no argument, and it is special because it is called when an object is introduced but is not initialized with any arguments.

In this example, the default constructor is called for `r3`. Note how `r3` is not even constructed with an empty set of parentheses - in fact, empty parentheses *cannot* be used to invoke the default constructor:
```cpp
Rectangle   r3;         // OK, default constructor called
Rectangle   r3();       // ERROR, default constructor NOT called – looks like a function prototype
```
This is because the empty set of parentheses above would make `r3` look like a function that takes no arguments and returns a value of type `Rectangle`.

#### Member variable initialization in Constructors

When a constructor is used to initialize the member variables of a class, the variables can be initialized directly within the body of the constructor, as we have been doing. For example, consider a class with the following declaration:
```cpp
class Rectangle {
  int   width, height;       // private member variables

 public:                     // public member functions
  Rectangle(int x, int y);   // constructor declaration
  int area() { return width * height;  }
};

// constructor definition
Rectangle::Rectangle (int x, int y) {
  width = x;
  height = y;
}
```
But it could also be defined using a member initialization list
```cpp
Rectangle::Rectangle (int x, int y) : width(x), height(y) {}
```
In this above case, the constructor does nothing more than initialize its member variables, hence it has an empty function body `{}`, but you could put other code inside the body if you wished.

For members of fundamental types e.g. `int`s, `float`s, `char`s etc. it makes no difference which of the two styles of constructor initialization you chose, however member variables whose type is another class *should* be initialized in the member initialization list. For example, consider a cylinder object:
```cpp
class Circle {
  double radius;
 public:
  Circle(double r) : radius(r) {}  // constructor using a member initialization list
  double getArea() {
    return radius*readius*PI;
  }
}

class Cylinder {
  Circle base;       // a member variable `base` which is a circle
  double height;
 public:
  Cylinder(double r, double h) : base(r), height(h) {}  // NOTE: constructor to initialize base
  double getVolume() {
    return base.getArea() * height;
  }
}
```
The `Cylinder` has a member variable `base` whose type is another class (a `Circle`).  Because objects of class `Circle` can oly be constructed with a parameter (i.e. the only available circle constructor requires an `int` argument), `Cylinder`'s constructor need to call `base`'s constructor, and the only way to do this is through a *member initializer list*.

#### Copy Constructors and References

A copy constructor is used to make copies of other class objects.  As an example, suppose we had a cylinder object `c1` in our code constructed like this
```cpp
Cylinder c1 (10, 20);  // call cylinder constructor
```
Now suppose we wanted to create another cylinder `c2` with exactly the same values as `c1`. Remember `c1` could have been changed by calling a member function after it was constructed, but we don't care, we simply want to copy its current value as it is now.

One way to create `c2` as an identical copy of `c1`, is to create `c2` first, read the current values of `c1` using functions like `getBase()` and `getHeight()` -- assuming they exist -- and copy them to `c2` using functions like `setBase(...)` and `setHeight(...)` -- again assuming they exist. However, what if those functions don't exist, or if the variables inside the cylinder were marked as `const` meaning their values cannot be changed after construction? If any of these were the case we would not be able to copy an existing object when creating a new one. This is exactly what a copy constructor is designed to overcome.

Note if you do not supply a copy constructor explicitly in your code, the compiler will create one for you automatically, and it will literally perform copying of all member variables in one object into their identical counterparts in the object, i.e. member-wise copying. Generally this default copy constructor works well, but what if the object we were copying had *pointers* in it?  We would end up copying the *pointer values* from the existing object to the new other -- this may be what we wanted but generally it's not, since both objects would end up have pointers pointing to the same "thing". If one of those objects were to destroy the "thing" in the future, what would the other one be left pointing to?

##### Writing a Copy Constructor

A copy constructor for the `Cylinder` class **must** have the following signature:
```cpp
Cylinder(const Cylinder &other)
```
Because it's a constructor, it will still have the same name as the class, i.e. `Cylinder`, it does not return anything (i.e. no `void` return type) and it takes a single argument which is a reference (the `&` operator) to an existing cylinder object `other`.  The implementation, using a member initialization list, could look like
```cpp
Cylinder::Cylinder (const Cylinder &other )  :
  base ( other.base ), height ( other.height ) {
  // anything else you want to do, put it here
}
```
We mark `other` as `const` to signal to users that we will not modify the original, only read data from it.  Note that since this constructor belongs to the `Cylinder` class, we do have direct access to `other`'s private member variables `base` and `height`, since `other` is also a `Cylinder`.

Assuming the existence of a `Cylinder` object `c1`, we could now write the following to "clone" that existing cylinder when creating a new cylinder "c2"
```cpp
Cylinder c2( c1 ) ;
```
The copy constructor is called because the argument `c1` is a `Cylinder`, so the call matches the copy constructor's method signature, i.e. construct `c2` based on `c1`.

##### References: The `&` operator in variable declarations

References don't exist in pure C, only C\+\+.  They are very similar to pointers, except
- they cannot be re-assigned
- they cannot point to nothing (i.e. `NULL`)
- to access members, you use the `.` operator instead of the `->` operator

Using a reference to a variable is exactly like using the variable itself.  Consider the following example:
```cpp
char c;        // a simple char
char *cptr;    // a pointer to char (i.e a char pointer)
cptr = c;      // initialize pointer to point to ‘c’
*cptr = 5;     // set the value of ‘c’ to 5 using the pointer with the ‘*’
```
Using references is sometimes more convenient
```cpp
char c;        // a simple char
char &cref;    // a char reference
cref = c;      // initialize reference to refer to ‘c’ (note NO ‘&’ in initialization)
cref = 5;	   // set the value of ‘c’ to 5 using the reference (note NO ‘*’)
```
The following demonstrates some of the differences:
```cpp
char d = 10;

cptr = &d;     // changes cptr to now point to d
*cptd = 10;    // c=5, d=10

cref = d;      // copies the value of d into whatever
               //   cref is referring to: c=10, d=10
               // There is no way to change what variable
               //   cref points to, it is equivalent to c
```
References are mostly used to either simply expressions, for example
```cpp
Circle &base = c1.getBase();  // get a reference to the cylinder's base
                              // so we can use it over and over again
base.setRadius(10);
int a = base.getArea();
...
```
or in function arguments to prevent copying of data
```cpp
void use_cylinder(Cylinder &c1);
```
In the above function definition, if we had omitted the `&` operator, then `use_cylinder(c1)` would actually construct a new cylinder every time the function is called, which could lead to significant overhead if `Cylinder` were a large object with many member variables.  Modern C\+\+ practice dictates
```cpp
// if c1 is used but not modified, use
void use_cylinder(const Cylinder &c1);  
// if c1 will be modified directly, use
void use_cylinder(Cylinder &c1);
// if a copy of c1 is needed, use
void use_cylinder(Cylinder c1);

// if c1 is allowed to be NULL (and you are going to check),
// but you are not going to modify it, use
void use_cylinder(const Cylinder* c1);
// if c1 is allowed to be NULL and (if not NULL) the
// contents will be modified directly, use
void use_cylinder(Cylinder* c1);
```
It may seem unnecessarily complicated which function declaration to use since you could probably write the function you want with any of them.  However, communicating intentions can be *extremely* important.  If another developer wants you use your function, they need to know what to expect.  What if he or she needs to keep their cylinder object unmodified (because it is used for other things as well)?  They need to know whether your function promises not to modify the argument, or if they instead need to make a protected copy of it, potentially adding overhead.

### Destructors

A C\+\+ class can also have a *destructor* function. It is declared with the same name as the class but with a `~` in front of its name. A destructor is automatically called whenever an object is destroyed (e.g. it goes out of scope) or is deliberately destroyed.

A destructor can be used to "tidy up" any loose ends after an object is destroyed. For instance, it could be used to close a file, or turn off an LED in an embedded application. It could also be used to release any memory back to the operating system that might have been allocated.

Like constructors, destructor functions do not have a return type; not even `void`. Neither do they take any arguments. For example, a destructor for the rectangle class would look like this:
```cpp
class Rectangle {
  int width, height;

 public:                    // public member functions
  Rectangle(int x, int y);  // constructor function
  ~Rectangle();             // destructor function (note the '~' in the name)
  int area() { return width*height; }
};

// code for destructor (note placement of ‘~’)
Rectangle::~Rectangle() {
  std::printf("Rectangle Destructor Called….\n") ;
}
```

### Pointers to Class Objects

Once a class has been introduced, a class name becomes a new valid type, so we can create pointers to that type of data. For example, the following statement declares that `p1` is a pointer to an object of class `Rectangle`. That is, it will eventually be initialized to point to an object in memory which is a `Rectangle`.
```cpp
Rectangle *p1;
```
As we have seen with plain data structures in C, the members of a `struct` can be accessed via a pointer using the arrow operator `->` (i.e. it's *pointing* to ...). The same principle applies when a pointer is used to access the members of a class.
```cpp
/****************************************************************************
* Example of using a pointer to a class object
****************************************************************************/
#include   <cstdio>  // std::printf

class Rectangle {
  int width, height;

 public:
  Rectangle(int x, int y) : width(x), height(y) { }
  int getArea(void) { return width * height }
};

int main() {
  Rectangle r1(3, 4);    // create a rectangle object
  Rectangle *p1;         // create a pointer to a rectangle

  p1 = &r1;              // make p1 point to object r1

  std::printf("r1’s area = %d\n", r1.getArea() );  // print area using r1 directly
  std::printf("r1’s area = %d\n", p1->getArea() ); // print area indirectly using pointer p1

    return 0;
}
```
We can *dereference* a pointer, getting at the underlying object, using the `*` operator. Note, however, the difference between:
```cpp
Rectangle r2 = *p1;    // create a copy of r1
Rectangle &r3 = *p1;   // create a reference to r1
```
The first *deferences* the pointer `p1` to get at the underlying rectangle `r1`, but then the `=` operator says "create a new rectangle  `r2` and set the contents equal to `r1`".  The second operation, however, creates a reference `r3` directly to `r1`, so the two are now treated as the same thing.  Changing `r2` does not affect `r1` at all, since it is a copy.  Changing `r3` **does** change `r1`, since they are the same object.  Confusing?  It can be, but this will get easier with practice.

### Dynamic Memory Allocations: operators `new` and `delete`

In the C language we could dynamically allocate storage for memory using library functions such `malloc`, `calloc`, `realloc` and `free`, defined in the header file `<stdlib.h>`. The functions are also available in C\+\+ (using the header `<cstdlib>` ) and can also be used to allocate and deallocate dynamic memory. But there is a nicer way to do this in C++ using operators `new` and `delete`.

For example in C\+\+, if we wanted to dynamically allocate memory for some variable, we could call the operator `new` to make that allocation. This operator returns a pointer to the data that we allocated storage for. As an example, we could allocate storage for a single new int like this
```cpp
int *p1 = new int ;
```
We could allocate storage for an array of 20 floats like this
```cpp
float *p2 = new float [20] ;
```
We could then use the pointers to manipulate the data as we would have done in C, e.g.
```cpp
*p1 = 5;    // set the integer that p1 points to, to the value 5
for ( int i = 0; i < 20; ++i ) {
	p2[i] = 1.25;  // set all elements in p2 to 1.25
}
```
When we have finished with the variables, we deallocate their storage using operator `delete` (previously we used `free` in C). For example, in C\+\+ we release the storage we allocated above like this:
```cpp
delete p1;      // release the single element pointed to by p1
delete [] p2;   // delete the array of elements pointed to by p2
```
This new technique for allocating and releasing memory is useful when we introduce classes. For example, we can easily dynamically allocate storage for class objects using
```cpp
// create storage for a rectangle, call its constructor
Rectangle *p1 = new Rectangle(3,4);

// allocate storage for an array of 10 rectangles.
// The default constructor called for each.
Rectangle *p2 = new Rectangle[10];

// free up the storage
delete p1;
p1 = nullptr;
delete [] p2;
p2 = nullptr;
```
Note the practice of setting the pointers to NULL after freeing the memory.  This ensures that you won't accidentally continue accessing the memory later on (since it would throw an exception).  In modern C\+\+ (C\+\+11 or higher), the keyword `nullptr` is used to represent an empty pointer.  Prior to this, it was common to use `NULL` or simply `0`.

### Class Inheritance

One of the main powers of classes is the ability to extend them, creating new classes by basing them existing ones.  This process, known as *inheritance*, involves a **base class** and a **derived class**: the derived class inherits all the member variables and functions of the base class (i.e. it is as if you had written them all again yourself).  You can also add new member variables and functions to the derived class. This is important because it leads to "re-use" of existing code: we can take something that somebody else may have written and add to it rather than re-inventing the wheel from scratch each time.

Inheritance also allows us to *override* (or modify) something we inherited from the base class. For example, we could write a new member function in our derived class which has the same name as the function in the base class, but which will do something different, i.e. we can tailor the behaviour of our new derived class to make it more suitable for our application.

For example, let's imagine a simple series of classes to describe two kinds of polygons: triangles and rectangles. These two polygons have certain common properties, such as the values needed to calculate their areas: they both could be described simply with a height and a width (or base) variable.

This could be represented in the world of classes with a class `Polygon` from which we would derive the two other shapes: `Triangle` and `Rectangle`:

![polygon inheritance]({{ site.url }}/assets/lectures/cplusplus/polygon_inheritance.png)

The `Polygon` (our **base**) class would contain the member variables and functions that are common for both rectangles and triangles, in our case the variables `width` and `height`.  `Rectangle` and `Triangle` would be new *derived* classes, with specific features that are unique to each type of polygon.

Classes that are derived from others *inherit* all the members of the base class. That means that if a base class like `Polygon` includes a member variable or function `A` and we derive a new class from it and then add a new member called `B`, to that derived class, then the derived class will contain both member `A` and member `B`. Nothing can be taken out during inheritance, it is not selective, you inherit everything from the base class. However, you can add to what you inherit.

#### Specifying Inheritance in C\+\+

New derived classes can be created using the following syntax:
```cpp
class Rectangle : public Polygon {
 ...
};

class Triangle : public Polygon {
 ...
}
```
For this course, we'll mostly stick with `public` inheritance (though if you look through the course library, you will see others).  This simply means that any method or variable that was public in `Polygon` will remain public in both `Rectangle` and `Triangle`.  The following is a definition for the base class:
```cpp
class Polygon {
  int width, height;   // private
  
 public:
  Polygon(int w, int h) : width(w), height(b) {}  // constructor
  int getWidth() { return width; }                // `getters'
  int getHeight() { return width; }
  void setWidth(int w) { width=w; }               // `setters'
  void setHeight(int h) { height=h; }
};
```
Next, we create our two derived classes and implement a `getArea()` function for each
```cpp
class Rectangle : public Polygon {
 public:
  Rectangle( int x, int y ) : Polygon(x,y) {}       // constructor calls base constructor
  int getArea() { return getWidth()*getHeight(); }  // call inherited functions
};

class Triangle : public Polygon {
 public:
  Triangle( int x, int y ) : Polygon(x,y) {}          // constructor calls base constructor
  int getArea() { return getWidth()*getHeight()/2; }  // call inherited functions
};
```
The `Rectangle` and `Triangle` constructors have to call the base class `Polygon` constructor to initialize its members.  Note that since both `width` and `height` were declared as `private` (the default), *only* `Polygon` has direct access to them.  Even though both `Rectangle` and `Triangle` both do have widths and heights internally, the two classes can only get access to them through `Polygon`'s accessor methods (`set...` and `get...`).  Derived classes are constructed from the bottom-up, initializing base class members first, so the base class constructor should be called in the member initialization list.

Even though we never explictly wrote a `getWidth()` or `getHeight()` method in either `Triangle` or `Rectangle`, both classes do have these methods.  They were inherited from the `Polygon` class, allowing us to re-use them, which is the whole point.  We don't want to have to re-invent the wheel every time.

#### Overriding members and Polymorphism

If a base class `X` had a member variable or function `A`, and if we derive a new class `Y` from `X` and add its own new member `A`, then the derived class will actually contain two members called `A`.  The one actually used depends on the specific type of the reference or pointer.  Consider the following:
```cpp
/*************************************************
* Example of Overriding (no Polymorphism)
*************************************************/
#include <cstdio>
class Animal {
 public:
  void speak() { std::printf("grunt\n"); }
};

class Dog : public Animal {
 public:
  void speak() { std::printf("woof\n"); }
};

int main() {
  Dog dog;      // creates a new dog
  dog.speak();  // "woof"
  
  Animal& dogref = dog;  // reference to dog
  dogref.speak();        // "grunt"
  return 0;
}
```
Here, even though `dog` and `dogref` are the same object (since `dogref` is a reference), they print different things when the method `speak()` is invoked.  This is because `Dog` actually has two member functions called `speak()`: one inherited from `Animal` and one defined directly in `Dog`.  When the method is called from a `Dog` reference or pointer, `Dog`'s version is called.  When the method is invoked from an `Animal` reference or pointer, `Animal`'s version is called.  This type of overriding is *rarely* used in practice.  It is often more useful to try to override the behaviour permantly if we know it is a dog.  That is where *polymorphism* comes in.

Polymorphism refers to the ability to permanently redefine methods for a derived class.  This would let the dog bark, for example, regardless of whether we treat it as a `Dog` or an `Animal`.  To accomplish this, we introduce the keyword `virtual`.  A *virtual function* is completely redefined in the derived class, so it only keeps one copy (the base class's version is erased).  This ensures that the derived class's version is called for an object regardless of the expression used.  Modifying our previous example slightly,
```cpp
/*************************************************
* Example of Overriding (WITH Polymorphism)
*************************************************/
#include <cstdio>
class Animal {
 public:
  virtual void speak() { std::printf("grunt\n"); } // virtual function
};

class Dog : public Animal {
 public:
  void speak() { std::printf("woof\n"); }          // overrides Animal's copy
};

int main() {
  Dog dog;      // creates a new dog
  dog.speak();  // "woof"
  
  Animal& dogref = dog;  // reference to dog
  dogref.speak();        // "woof" (SAME AS BEFORE)
  return 0;
}
```
Once a functon is declared `virtual` in a base class, it remains virtual for all derived classes.  For example, if we create a new class,
```cpp
class ScoobyDoo : public Dog {
  void speak() { std::printf("ruh roh\n") } // overrides Dog's and Animal's version
};

int main() {
  SkoobyDoo bydoo;
  Animal* animal = &bydoo;  // Animal pointer to Scooby
  animal->speak();          // "ruh roh"
}
```
Even though `Dog` didn't explicitly declare `speak()` as virtual, it was still overridden by `ScoobyDoo`'s version.  The method remains virtual because `Animal` declared it as such.

**Bottom line:** if you're planning to override a function (which is usually by design), declare it as `virtual` in the base class.  That way, any class that overrides it is guaranteed to call its own version, regardless of how that variable is accessed.

### Exercises

In solid modelling and computer graphics, three-dimensional objects are often represented by a *polygon mesh*, which consists of vertices, edges, and faces.
![gear]({{site.url}}/assets/lectures/cplusplus/gear.png)<br/>
*Gear triangulated model, modified from Christopher Spicer's [16-Tooth Spur Gear](https://3dexport.com/free-3dmodel-16-tooth-spur-gear-139745.htm).*

In this exercise, we are going to build a basic mesh representation.

#### Part A

First we will contruct a few basic classes that will allow us to represent a triangular mesh (i.e. all faces are triangles).
<ol>
  <li>Create a <code>Vertex</code> class that has
    <ul>
      <li>a <b>private</b> member variable <code>idx</code> for storing an index value</li>
      <li>three <b>public</b> member variables <code>x,y,z</code> for storing position</li>
      <li>two overloaded <em>constructors</em>:
        <ul>
          <li>one that sets the index alone, initializing the position to zero</li>
          <li>the other that sets both the index and position</li>
        </ul>
      </li>
      <li>a <b>public</b> method for getting the vertex index</li>
      <li>a <b>public</b> method for setting the vertex position</li>
    </ul><br/>
    <img src="{{ site.url }}/assets/lectures/cplusplus/vertex_class_diagram.png" alt="vertex class diagram"/><br/>
    <em>Class Diagram for the</em> <code>Vertex</code> <em>class</em><br/><br/>
  </li>
  <li>Create a <code>TriFace</code> class that has
    <ul>
      <li>a <b>private</b> member variable <code>idx</code> for storing an index value</li>
       <li>three <b>public</b> member variables <code>v0,v1,v2</code> for storing <code>Vertex</code> <b>pointers</b> that make up the face (ordered counter-clockwise around the outward-pointing normal using the right-hand rule)</li>
       <li>a single constructor which sets the face index and the three vertices</li>
       <li>a <b>public</b> method for getting the face index</li>
       <li>a <b>public</b> method for getting the number of vertices in the face (should be 3)</li>
    </ul><br/>
    <img src="{{ site.url }}/assets/lectures/cplusplus/triface_class_diagram.png" alt="triangular face class diagram"/><br/>
    <em>Class Diagram for the</em> <code>TriFace</code> <em>class</em><br/><br/>
  </li>
  <li>Create a <code>PolygonalMesh</code> class for storing a collection of vertices and faces.  The internal details are left to you, but it must support <b>public</b> functions to:
    <ul>
      <li>get the number of vertices</li>
      <li>get the number of faces</li>
      <li>get a <b>pointer</b> to the <em>i</em> th vertex (should correspond to vertex with index <em>i</em> )</li>
      <li>get a <b>pointer</b> the <em>i</em> th face (should correspond to face with index <em>i</em> )</li>
    </ul><br/>
    <img src="{{ site.url }}/assets/lectures/cplusplus/polygonalmesh_class_diagram.png" alt="polygonal mesh class diagram"/><br/>
    <em>Class Diagram for the</em> <code>PolygonalMesh</code> <em>class</em><br/><br/>
  </li>
</ol>

To test your code, try creating a few sample mesh instances.  You may find it convenient to read-in a mesh from a file rather than code-up all the vertex positions and faces manually.  A very basic "wavefront" object file (.obj) looks like this:
```
v -0.241144 0.007238 -0.039053
v -0.241144 0.004954 -0.016076
v -0.243538 0.007238 -0.016076
v -0.241144 0.004971 0.016076
v -0.243516 0.007238 0.016076
...
f 1 2 3
f 2 4 5
f 3 2 5
...
```
Lines that define a vertex start with `v`, and give its 3D coordinates. Vertices are numbered sequentially starting with index `1`.  Lines that define a face start with `f` and list the vertices that make up the face, so `f 1 2 3` tells us to create a single triangular face using the first three vertices above.  Here are sample wavefront files for a [cube]({{site.url}}/assets/lectures/cplusplus/cube.obj), [sphere]({{site.url}}/assets/lectures/cplusplus/sphere.obj), and the [gear]({{site.url}}/assets/lectures/cplusplus/gear.obj).  See if you can re-create the files from your `PolygonalMesh` class, and verify the geometry is the same (either by comparing the files themselves, or by loading your newly created file in something like [MeshLab](http://www.meshlab.net/) or Windows 3D Builder).

#### Part B

Not all faces in a polygon mesh need to be triangles -- otherwise it would just be a *triangular* mesh, not a *polygon* mesh.  We will add a second type of face: the quadrilateral-face.  This new face type will also have an index number and support the `numVertices()` query, so we will use abstraction and inheritance to build a class hierarchy.

 <img src="{{ site.url }}/assets/lectures/cplusplus/face_class_diagram.png" alt="face class diagram"/><br/>
    <em>Class Diagram for the</em> <code>Face</code> <em>class hierarchy</em><br/>

1. Create a new **base class** called `Face` that implements the shared functionality.  In the class diagram above, note that we did not include `v0,v1,v2` in the base class.  We technically could, but this may give the misleading impression that all faces are triangles.  The base class should only contain what might be expected from *all* faces.
2. Adjust your code for `TriFace` to extend `Face`
3. Create a new `QuadFace` class that has a fourth vertex pointer `v3`
4. Add a new method to `Face` and its subclasses, `Vertex* getVertex(int i)`, that returns the *i* th vertex of the face.  In the base class, you can either give a basic definition that returns a null pointer (`nullptr` in C++11 or `NULL` otherwise), or you can define it as what's called a *pure virtual function*
  ```cpp
  virtual Vertex* getVertex(int i) = 0;
  ```
  This will make the `Face` class *abstract*.  Since no definition of `getVertex` exists in `Face` (we have essentially cleared it by setting it to zero), nobody can explicitly create a `Face` directly.  Instead, they must create one of the subclasses which have overridden the method to provide an implementation.
5. Change your `PolygonalMesh` class to use the base `Face` class so it can hold both triangular and quadrilateral faces.

This exercise demonstrates another advantage of classes and inheritance: *abstraction*.  From the point-of-view of the mesh, it doesn't really matter if a face is triangular or quadrilateral.  Internally, it can store both types in an array that holds instances of type `Face`.  We can get all the information we need about either type of face using only the methods in the base class.  The sub-classes only provide implementation-specific details.

## Acknowledgements

Some of this material is derived from the [cplusplus.com tutorials on classes](http://www.cplusplus.com/doc/tutorial/classes/) (© cplusplus.com, 2000-2017).  The gear model used in the exercises was modified from Christopher Spicer's [16-Tooth Spur Gear](https://3dexport.com/free-3dmodel-16-tooth-spur-gear-139745.htm).

## Additional Resources

Have a look at the following resources to learn a bit more.  The C\+\+ tutorials are a great start for beginners, and goes through many of the topics covered here.  For the brave or more advanced programmer, have a look at the Guru-of-the-Week blog, which contains an interesting set of problems about the nitty-gritty details of the C\+\+ language that can help make you a guru in no time.
- C++ Language Tutorials:  [http://www.cplusplus.com/doc/tutorial/](http://www.cplusplus.com/doc/tutorial/)
- Herb Sutter's Guru-of-the-Week: [https://herbsutter.com/gotw/](https://herbsutter.com/gotw/), [http://www.gotw.ca/gotw/](http://www.gotw.ca/gotw/)



