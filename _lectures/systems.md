---
layout: lecture
title:  Introduction to System Software Engineering
date:   2017-09-07 17:50:00
authors: [Paul Davies, C. Antonio Sánchez]
categories: [lectures, systems, real time]
published: false

---

# Introduction to System Software Engineering
{:.no_toc}

**Authors:** Paul Davies (UBC), C. Antonio Sánchez (UBC)

{% include toc.html %}

## Learning Goals

In this lecture, we discuss *System Software*, and the constraints involved in *Real-Time Systems*.  By the end of this lecture, you should be able to:

- Define a *system*, *system software*, and *system software engineering*
- Distinguish between *hard* and *soft* time constraints
- Distinguish between *event-driven* and *time-driven* systems and give some examples
- Describe the difference between *interrupts* and *polling* for detecting and handling events
- List advantages and disadvantages of both *interrupts* and *polling*
- Discuss why testing real-time systems can be challenging

## System Software Engineering

### What is a *System*?

**Etymology:** from the Greek [*σύστημα*](http://www.perseus.tufts.edu/hopper/text?doc=Perseus%3Atext%3A1999.04.0057%3Aentry%3Dsu%2Fsthma), which in Latin became *systēma*, meaning "whole compounded of several parts or members".[^fnsystem]

There are two important factors here: "several parts", which says that a system is made up of two or more somewhat independent components (i.e. are separate enough that they can be considered distinct entities); and "whole", which means that the components are designed to work together for some common purpose or to perform a unified task.

### What is *System Software?*

A simple definition would be that it is software that runs or controls systems.  This is in contrast to a *software system* which would be a system of software components.

The term *system software*, however, has taken on a bit of a broader meaning.  The software universe is divided into two categories: **system software** and **application software**.  The line between the two isn't always clear, but in general

<dl>
<dt><b>System Software:</b></dt>
<dd>is system-centered.  It typically runs in the background to control hardware or other software components, fairly autonomously.  System software may interact with the user, but that interaction is not the main use-case.  Examples include operating systems, control systems, system utilities for performing maintenance, etc.</dd>
<dt><b>Application Software:</b></dt>
<dd>is user-centered.  It performs a specific task directly for the user, requested by the user.  Examples include word processors, video games, basic web-browsers, photo editors, etc.</dd>
</dl>

These definitions can get a bit murky when it comes to some modern web-browsers that support and run other internal-applications (acting more like an operating system), and are themselves deeply integrated into the OS.  When the US government brought an [anti-trust suit against Microsoft](https://en.wikipedia.org/wiki/United_States_v._Microsoft_Corp.) for bundling Internet Explorer with Windows and preventing it from being uninstalled, Microsoft argued that IE was an essential component of the OS, making it part of the underlying system (i.e. part of the system software).

Whether or not IE is or isn't actually system software isn't so important for our purposes.  What *is* important is that writing system software has some implications and potentially introduces some constraints.  Because the components of a system are fairly autonomous, each will require software that is somewhat self-contained.  However, in order to work together to perform some grand function, the various parts will need to be able to communicate with each other, and will need to synchronize their actions.  We will spend a good chunk of this course discussing these communication and synchronization mechanisms.

### What is *Engineering*?

**Etymology:** the word *engineer* has the same Latin roots as the word *ingenuity*: *ingeniare*, which means "to contrive or devise", and *ingenium*, which means "cleverness"[^fneng].  *Engine* is also part of the same family, described as a product of ingenuity.

Engineering now refers to the use of knowledge in order to purposefully design, develop and build a product (be it a physical product, like an engine, or a work product like software).

*System Software Engineering* is therefore the deliberate design of system software.  Design requires planning.  Just as there are tools and established methods for designing bridges and engines, there are also tools and methods for designing software and software systems. This planning becomes extremely important when we start dealing with large and real-world systems.  Any software controlling critical systems needs to be proven to work on paper, first, and systematically tested before being released for its intended application.  We will spend the other good chunk of this course discussing these design tools and methodologies, how to communicate those designs, and how to properly test our software.

## Real-Time Systems

A precise definition of a *real-time system* is hard to pin down.  They are most often described by a set of characteristics or properties, such as the ability to react to inputs within a certain timeframe ,and generate a significant or meaningful outcome.  But how short must the time-frame be to be considered real-time? And how meaningful must the response be?  Five popular definitions are:

1. A real-time system is generally a *controlling system*, often *embedded* into equipment so that its existence is not obvious.  It takes in *information* from its environment, *processes it* and *generates a response*.
2. A real-time system *reacts*, *responds*, and *alters its actions* to *affect* the *environment* in which it is placed.
3. A real-time system implies that there is something *significant* and important about its *response time*.
4. A real-time system has a *guaranteed*, *deterministic* *worst-cast response time* to an event under its control.
5. A real-time system is one where the correct answer at the wrong time is the wrong answer.

There seems to be an emphasis on response time, but note that there are no explicit mention that the response must be generated instantaneously, or even within a specific time period.  That is because whether or not a system is considered real-time is always dependent on **context** and the particular application at hand.

For example, consider real-time audio processing.  For playing back one's own voice, latency must be limited to below 15ms to prevent the delay from causing a significant distraction.  For telephone or VoIP conversations, a delay of up to 200ms is considered acceptable, after which call quality drops significantly.  For audio-video delay, it is recommended that the the audio should not preceed the video by more than 40ms or lag by more than 60ms.  "Real-time", in this case, means to process and output the audio signal before the delay causes significant perceptual differences.

On the other-hand, consider a real-time temperature and humidity monitoring and control system.  Does a temperature need to be recorded and adjusted every 200ms?  How about 2s?  It's doubtful a system could even change the temperature of an environment that quickly.  What is the system for?  If it's controlling the humidity and temperature of a home, then it need only respond fast enough to maintain an acceptable level of comfort.  If it's for protecting valuable art pieces or computer servers, then the system must respond fast enough to prevent potential damage.

In both the audio processing and temperature control examples, whether or not a system is considered "real-time" depends on the context of the application.  The exact threshold is set to prevent either undesireable or unacceptable outcomes.

### Hard versus Soft Real-Time Systems

A *hard* real-time system is one in which a failure to satisfy the specified *worst-case response time* leads to an overall *system failure*.  In other words, there is a *hard limit* on response time.  Specifications will often describe maximum and minimum acceptable bounds.  These systems are *time-critical*.  A failure may cause damage, significant cost, or serious injury.  Our temperature control system protecting computer equipment, for example, can be considered a hard real-time system.  Other examples include heart monitors, Nuclear power station control systems, and Tesla's autopilot feature.

In a *soft* real-time system, failure to meet the response time limits leads to a *degredation* of the system, but not necessarily an overall failure.  Specifications will often describe suggested or expected response times.  Think of the telephone application.  Beyond a 200ms delay, call quality will drop, but there are no damaging consequences.  Of course, if the response time is sufficiently over the soft limit, it may still result in failure in that the system no longer functions acceptably.  Who would put up with a phone call where there is a 2 minute delay?  Other examples include elevator control systems, ATMs, and household thermostats.

### Event-Driven verses Time-Driven Systems

Real-time systems can also be classified in terms of how they get their inputs and generate responses.  We will consider two types:

<dl>
<dt>Event-Driven</dt>
<dd>An input sensor is responsible for detecting the occurence of an <em>event</em> that impacts the system's success, and <em>triggers</em> a response</dd>
<dt>Time-Driven</dt>
<dd>The system processes inputs and reacts at <em>specified times</em>, such as at a periodic interval.</dd>
</dl>

#### Event-Driven Systems

In event-driven systems, there are two ways for detecting the occurence of an event: *interrupts*, or *polling*.
<br/><br/>

**Interrupts**

With interrupts, the sensor generates an *interrupt signal* that tells the processor there is an important event to take care of.  The CPU usually responds by suspending its current thread, saving its state, then calling an *interrupt handler* (or *interrupt service routine*) to deal with the event.  Once the handler has finished, the processor can load its previous state and resume the original thread.

Advantages:

- The system is notified immediately of the event, so can be handled immediately
- This leads to more deterministic response times
- Interrupts can be prioritized
- Response time is independent of the number of sensors (as long as there aren't so many expected interrupts that it overloads the system)
- Frees up processing power, since CPU is free to do other things between interrupts

Disadvantages:

- More complex hardware is needed (sensors need to be able to trigger interrupts, additional hardware for prioritizing before CPU is notified, system-level handlers used for triggering the user's service routines, etc.)
- Difficult to test and debug since timings of interrupts is unpredictable, and the interrupt handler and main code run *asynchronously*.
<br/><br/>

**Polling**

With polling, the main program continuously probes the sensors to check their status and see if there is an event that needs to be handled.  Upon detecting an event, the program can then decide whether to handle in from within its main thread, or to launch a separate asynchronous thread to deal with it.

Advantages:

- Simpler code and no need to integrate complex interrupt service routines.
- Easier to debug since event detection is predictable (it is now *synchronous*).
- More flexibility in handling events.

Disadvantages:

- Sensor events may be missed (e.g. if they only persist briefly, or if multiple events are triggered by the same sensor in rapid succession).
- Difficult to prioritize events.
- Adding more sensors degrades performance and response times, since more time is spent querying their status and events may only be detect after some delay.
- Consumes a lot of CPU time, particularly if events are few and far between, as the CPU will keep unnecessarily looping through and checking each of the sensors.

#### Time-Driven Systems

In time-driven systems, the system is responsible for ensuring that certain actions occur at *specified times* or *periodic intervals*.  Execution is usually controlled by a hardware *timer* that can be tuned to run with a specific resolution.  For example, consider a temperature logging device that samples the current temperature every 50ms, saves its memory buffer to a file every 5s, and sends the log file to a remote server every 5min.

For time-driven systems, the designer usually decomposes the set of activities into a number of smaller *threads* or *processes* and ensures that the operating system *scheduler* runs the tasks at the presecribed times (assuming that it is possible).

### Testing Real-Time Systems

Testing real-time systems can be challenging because they often involve *asynchronous* and unpredictable "real-world" events.  How are we to know what an elevator is going to be doing 5 seconds after we push the button?  What is an autopilot going to be busy with 20 minutes into a flight?

Exhaustively testing such systems is both expensive and time-consuming.  We would have to run through every possible sequence of events, with all possible timings, and check the system's response.  But with each new event, the number of required tests grows exponentially.  Add to this the variable timings, and subjecting the system to all these combinations becomes impossible.

If the system *does* fail a test, it may even be difficult to reproduce the conditions to figure out *how* and *why* the system failed.  This makes debugging a big challenge.  We can't use the same break-point and step procedure as regular single-threaded programs, as this will interfere with timings and sequences.  Adding print-statements or logging can also interfere with timings, changing them just enough that the bug is no longer reproducible.

Unfortunately, because of the costs and challenges with testing, real-time software systems are often released without adequately exhaustive testing.  This can lead to some pretty dangerous situations and has resulted in serious injury and death:

- [Air-traffic control: lost contact countown](http://spectrum.ieee.org/aerospace/aviation/lost-radio-contact-leaves-pilots-on-their-own) 
- [Therac-25: accidental radiaton overdose](http://hackaday.com/2015/10/26/killed-by-a-machine-the-therac-25/)

We will discuss software testing more in a future lecture.  For now, it is enough to recognize that testing is important, and can be difficult for system software, particularly with real-time systems.  We will learn some approaches for handling this complexity.  Software testing, just like the system software itself, must be carefully designed and systematically executed.

## Additional Resources

Check out these resources for a few more notes on systems:

- [http://www.embedded.com/electronics-blogs/beginner-s-corner/4023859/Introduction-to-Real-Time](http://www.embedded.com/electronics-blogs/beginner-s-corner/4023859/Introduction-to-Real-Time)

## References

[^fnsystem]: [https://en.wikipedia.org/wiki/System](https://en.wikipedia.org/wiki/System)
[^fneng]: [https://en.oxforddictionaries.com/definition/engineer](https://en.oxforddictionaries.com/definition/engineer)

