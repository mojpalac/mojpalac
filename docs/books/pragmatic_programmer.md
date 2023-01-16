# Pragmatic programmer

## 6. Concurrency

definition:<br>
_Concurrency_ is when the execution of two or more pieces of code **act** as if they run at the same time.<br>
_Parallelism_ is when they **do** run at the same time.

concurrency is a software mechanism, and parallelism is a hardware concern. If we have multiple processors, either
locally or remotely, then if we can split work out among them we can reduce the overall time things take.

### 33. Breaking Temporal Coupling

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