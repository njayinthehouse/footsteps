
# P: Safe Asynchronous Event-Driven Programming

***Source***: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-8.pdf

## Possible Prerequisites

* [Delay-Bounding](./depth-bounding.md) : [Delay-Bounded Scheduling](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/popl198ap-emmi.pdf)

## Contributions

* A design for P, A DSL to program asynchronously at a higher level of abstraction than event handlers
* Operational semantics for P
* A compiler and runtime that allows P programs to run as [Kernel Mode Driver Framework](https://en.wikipedia.org/wiki/Kernel-Mode_Driver_Framework) device drivers
* How to validate P programs using delay-bounded scheduling 
* A novel delaying scheduler that, by default, tries to schedule events according to their causal order
* Case study: P in the USB stack in Windows 8

## Overview

* A P program is a collection of machines communicating asynchronously through events
* Ghost machines are used only during verification

```
type DefferedSet = {Event}
type ActionHandlers = {(Event, Action)}
type State = (n: StateName, d: DeferredSet, a: ActionHandlers, s: EntryStatement)
```

* Entry statement -- gets evaluated as soon as the state is entered

### Deferred Events and Action Handlers

* Event sent to machine -- FIFO queued
* * It is possible to change the order in which events get delivered
* Receiving event -- dequeue returns the first event *that is not in the deferred set*

```
send : Event -> Machine -> EventQ
send e m = enqueue e (eventQueue m)

receive : Machine -> Option Event
receive m = dequeue' (eventQueue m)
   where dequeue' [] = Nothing
         dequeue' q@(e :: es) = if e in (deferredSet . currentState) m then dequeue' es else Some e
```

### Step and Call Transitions

* Step -> simple edge
* Call => double edge -> subroutine-like abstraction

### Unhandled Events

* No unhandled non-deferred events allowed
* No universally deferred events allowed

### Environment Modeling

* `if * then doSomething` -> * represents both true and false, and non-deterministically performs both
* Ghost machines -> used for modeling, do not generate code

