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

## 7 Concurrency
