# Pragmatic programmer

## 6 Concurrency

definition:<br>
_Concurrency_ is when the execution of two or more pieces of code **act** as if they run at the same time.<br>
_Parallelism_ is when they **do** run at the same time.

concurrency is a software mechanism, and parallelism is a hardware concern. If we have multiple processors, either
locally or remotely, then if we can split work out among them we can reduce the overall time things take.

### 33 Breaking Temporal Coupling

temporal coupling - coupling in time. Method A must always be called before method B, only one report can be run at a
time, you must wait for the screen to redraw before the button click is received. Tick must happen before tock.
This is not very flexible and not very realistic.

Allowing concurrency and think about decoupling of any time or order dependencies.
Result: systems that are easier to reason about, that potentially respond faster and more reliably.

#### Looking for concurrency

In many projects we'd like to find out what **can** happen at the same time and what **must** in a strict order.
One way to do is by _activity diagram_.

activity diagram - set of actions (drawn as rounded boxes). The arrow leads an action leads to either:

- another action (which can start after the first action completes)
- thick line called a _synchronization bar_ <br>

once all actions leading into a synchronization bar are complete you can then proceed along any arrows leaving the bar,
An action with no arrows leading into it can be started at any time.
Activity diagram is used to maximize parallelism by identifying activities that could be performed in parallel but
aren't.

For instance, we may be writing the software for a robotic piña colada maker.<br>
We’re told that the steps are:

1. Open blender
1. Open piña colada mix
1. Put mix in blender
1. Measure 1/2 cup white rum
1. Pour in rum
1. Add 2 cups of ice
1. Close blender
1. Liquefy for 1 minute
1. Open blender
1. Get glasses
1. Get pink umbrellas
1. Serve

![activity diagram](../../resources/images/activity_diagram.png)

the top-level tasks (1, 2, 4, 10, and 11) can all happen concurrently, up front. Tasks 3, 5, and 6 can happen in
parallel later.
When we look at the activities, we realize that number 8, liquify, will take a minute. During that time, our bartender
can get the glasses and umbrellas (activities 10 and 11) and probably still have time to serve another customer.

### 34. Shared State is Incorrect State

Customer is in restaurant asks server if there is a pie left, he looks, sees in the display that there is one left piece
and
confirms. Customer orders the pie.
Same thing happens on the other side of the restaurant at exact time.
One of the customers will be disappointed.

![](../../resources/images/non_atomic_diner_example.png)

Both waiters operate concurrently (in real life in parallel).
The problem above is shared state. Both waiters when executes _display_case.pie_count()_ they copy the value from the
display into their own memory. If the value in the display case changes their memory (which is used to make decision) is
now out of date.
Solution? make the operation atomic.

Semaphore is a _thing_ that only one person can own at a time. (lock/unlock claim/release)
In above example it can be used to decide who can access pie case.

```
case_semaphore.lock() 
 if display_case.pie_count > 0
    promise_pie_to_customer()
    display_case.take_pie()
    give_pie_to_customer()
 end
case_semaphore.unlock() 
```

This approach above works and solve mentioned issue, but there is another problem.
This approach as long as every developer will use the semaphore, it that's not the case we are in the same place as
before.

**Make the Resource Transactional**
We can centralize that control by checking and getting pie in one call:

```
def get_pie_if_available()
    @case_semaphore.lock() 
    try {
        if @slices.size > 0
            update_sales_data(:pie) 
            return @slices.shift
        else
            false
        end
        }
    ensue {
         @case_semaphore.unlock()
    }
    end
```

**Random Failures Are Often Concurrency Issues**

### 35 Actors and Processes

- An _actor_ is an independent virtual processor with its own local (and private) state. Each actor has a mailbox. When
  a
  message appears in the mailbox and the actor is idle, it kicks into life and processes the message. When it finishes
  processing, it processes another message in the mailbox, or, if the mailbox is empty, it goes back to sleep.

When processing a message, an actor can create other actors, send messages to other actors that it knows about, and
create a new state that will become the current state when the next message is processed.

- A _process_ is typically a more general-purpose virtual processor, often implemented by the operating system to
  facilitate concurrency. Processes can be constrained (by convention) to behave like actors, and that’s the type of
  process we mean here.

**Actors Can Only Be Concurrent**<br>
There are a few things that you won’t find in the definition of actors:

- There’s no single thing that’s in control. Nothing schedules what happens next, or orchestrates the transfer of
  information from the raw
  data to the final output.
- The only state in the system is held in messages and in the local state of each actor. Messages cannot be examined
  except by being read by their recipient, and local state is inaccessible outside the actor.
- All messages are one way—there’s no concept of replying. If you want an actor to return a response, you include your
  own mailbox address in the message you send it, and it will (eventually) send the response as just another message
  to that mailbox.
- An actor processes each message to completion, and only processes one message at a time.

As a result, actors
execute concurrently, asynchronously, and share nothing. If you had enough physical processors, you could run an actor
on each. If you have a single processor, then some runtime can handle the switching of context between them. Either way,
the code running in the actors is the same.

#### Tip 59 Use Actors for concurrency without Shared State

### Topic 36 Blackboards

Blackboard is a common (shared?) space where multiple independent actors/processes/agebts can access the stored data in
a form of _laissez faire_ concurrency,

## 7 While You Are Coding

### 37 Listen to Your Lizard Brain

#### Tip 61 Listen to Your Inner Lizard

1. First, stop what you’re doing.
2. Give yourself a little time and space to let your brain organize itself.
3. Stop thinking about the code,
4. do something that is fairly mindless for a while, away from a keyboard. Take a walk, have lunch,
   chat with someone. Maybe sleep on it. Let the ideas percolate up through the layers of your brain on their own: you
   can’t force it.
5. wait for a "ha!" moment.

If that doesn't work, try to explain the code to someone (even to rubber duck).

If still you have a problem, it's time for prototyping.

### 38 Programming by Coincidence

Why should you take the risk of messing with something that’s working? <br>
Well, we can think of several reasons:

- It may not really be working—it might just look like it is.
- The boundary condition you rely on may be just an accident.
  In different circumstances (a different screen resolution, more CPU cores), it might behave differently.
- Undocumented behavior may change with the next release of the library. Additional and unnecessary calls make your code
  slower. Additional calls increase the risk of introducing new bugs of their own. For code you write that others will
  call, the basic principles of good modularization and of hiding implementation behind small, well-documented
  interfaces can all help.

tip: never store a phone number in numeric field

### 39 Algorithm Speed

#### Big-O Notation - mathematical way of dealing with approximations.

highest-order term will dominate the value as increases, the convention is to remove all low-order terms, and not to
bother showing any constant multiplying:

O(n^2/2 + 3n) == O(n^2) == O(n^2)

```
O(1) - Constant (access element in array, simple statements)
O(lg n) - Logarithmic (binary search). The base of the logarithm doesn’t matter, so this is equivalent.
O(n) - Linear (sequential search)
O(n lg n) - Worse than linear, but not much worse. (Average runtime of quicksort, heapsort)
O(n^2) - Square law (selection and insertion sorts)
O(n^3) - Cubic (multiplication of two  matrices)
O(C^n) - Exponential (traveling salesman problem, set partitioning)
```

![](../../resources/images/runtime_of_various_algorithms.png)

### 40 Refactoring

Software development is like gardening -You plant many things in a garden according to an initial plan and conditions.
Some thrive, others are destined to end up as compost. You may move plantings relative to each other to take advantage
of the interplay of light and shadow, wind and rain. Overgrown plants get split or pruned, and colors that clash may get
moved to more aesthetically pleasing locations. You pull weeds, and you fertilize plantings that are in need of some
extra help. You constantly monitor the health of the garden, and make adjustments (to the soil, the plants, the layout)
as needed.

Refactoring [Fow19] is defined by Martin Fowler as a: disciplined technique for restructuring an existing body of code,
altering its internal structure without changing its external behavior.

_Time pressure is often used as an excuse for not refactoring. But this excuse just doesn’t hold up: fail to refactor
now, and there’ll be a far greater time investment to fix the problem down the road—when there are more dependencies to
reckon with. Will there be more time available then? Nope._

When explaining think of the code that needs refactoring as “a growth.” Removing it requires invasive surgery. You can
go in now, and take it out while it is still small. Or, you could wait while it grows and spreads—but removing it then
will be both more expensive and more dangerous. Wait even longer, and you may lose the patient entirely.

Martin Fowler offers the following simple tips on how to refactor without doing more harm than good:

1. Don’t try to refactor and add functionality at the same time.
2. Make sure you have good tests before you begin refactoring. Run the tests as often as possible. That way you will
   know quickly if your changes have broken anything.
3. Take short, deliberate steps:
    - move a field from one class to another,
    - split a method,
    - rename a variable.

Refactoring often involves making many localized changes that result in a larger-scale change. If you keep your
steps small, and test after each step, you will avoid prolonged debugging.

### 41 Test to Code

_We believe that the major benefits of testing happen when you think about and write the tests, not when you run them._

Why? Because it allows you to understand the under the hood requirements of feature (e.g. parameters)

Use TDD, but avoid:

- They spend inordinate amounts of time ensuring that they always have 100% test coverage.
- They have lots of redundant tests. For example, before writing a class for the first time, many TDD adherents will
  first write a failing test that simply references the class’s name. It fails, then they write an empty class
  definition and it passes. But now you have
  a test that does absolutely nothing; the next test you write will also reference the class, and so it makes the first
  unnecessary. There’s more stuff to change if the class name changes later. And this is just a trivial example.
- Their designs tend to start at the bottom and work their way up. (TIP 68, Build End-to-End, Not Top-Down or Bottom Up)

Build small pieces of end-to-end functionality, learning about the problem as you go. Apply this learning as you
continue to flesh out the code, involve the customer at each step, and have them guide the process.

The basic cycle of TDD is: Decide on a small piece of functionality you want to add. Write a test that will pass once
that functionality is implemented. Run all tests. Verify that the only failure is the one you just wrote. Write the
smallest amount of code needed to get the test to pass, and verify that the tests now run cleanly. Refactor your code:
see if there is a way to improve on what you just wrote (the test or the function). Make sure the tests still pass when
you’re done. The idea is that this cycle should be very short: a matter of minutes, so that you’re constantly writing
tests and then getting them to work.

### 42 Property-Based Testing

invariants, things that remain true about some piece of state when it’s passed through a function. For example, if you
sort a list, the result will have the same number of elements as the original—the length is invariant.

#### Tip Use property based tests to validate your assumptions
