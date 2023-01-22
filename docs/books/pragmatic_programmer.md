# Pragmatic programmer

## 6 Concurrency

definition:<br>
_Concurrency_ is when the execution of two or more pieces of code **act** as if they run at the same time.<br>
_Parallelism_ is when they **do** run at the same time.

concurrency is a software mechanism, and parallelism is a hardware concern. If we have multiple processors, either
locally or remotely, then if we can split work out among them, we can reduce the overall time things take.

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
When we look at the activities, we realize that number 8, liquefy, will take a minute. During that time, our bartender
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

Blackboard is a common (shared?) space where multiple independent actors/processes/agents can access the stored data in
a form of _laissez-faire_ concurrency,

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
  slower. Additional calls increase the risk of introducing new bugs of their own. For code, you write that others will
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
  definition, and it passes. But now you have
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

There are also code invariants, things that remain true about some piece of state when it’s passed through a function.
For example, if you sort a list, the result will have the same number of elements as the original—the length is
invariant. Once we work out our contracts and invariants (which we’re going to lump together and call properties) we can
use them to automate our testing. What we end up doing is called property-based testing.

```python
from hypothesis import given
import hypothesis.strategies as some 

@given(some.lists(some.integers()))
def test_list_size_is_invariant_across_sorting(a_list):
    original_length = len(a_list)
    a_list.sort() 
    assert len(a_list) == original_length 
    
@given(some.lists(some.text()))
def test_sorted_result_is_ordered(a_list):
    a_list.sort()
    for i  in range(len(a_list) - 1): 
        assert a_list[i] <= a_list[i + 1]

Thomas, David; Hunt, Andrew. Pragmatic Programmer, The (pp. 394-395). Pearson Education. Kindle Edition. 
```

### 43 Stay Safe Out There

#### Security Basic Principles

Pragmatic Programmers have a healthy amount of paranoia. We know we have faults and
limitations, and that external attackers will seize on any opening we leave to compromise our systems. Your particular
development and deployment environments will have their own security-centric needs, but there are a handful of basic
principles that you should always bear in mind:

1. Minimize Attack Surface Area
2. Principle of The Least Privilege
3. Secure Defaults
4. Encrypt Sensitive Data
5. Maintain Security Updates

### 44 Naming Things

whenever you create something, you need to pause and think “what is my motivation to create this?”

It’s important that everyone on the team knows what project jargon means, and that they use them consistently.

1. encourage a lof of communication e.g. Pair programming, huddles
2. Keep project glossary, listing the terms that have special meaning ot the team (e.g. on confluence)

## 8 Before The Project

### 45 The Requirements Pit

### Tip 76 Programmers Help People Understand What They Want

we annoy people by looking for edge cases and asking about them.

_- You: We were wondering about the $50 total. Does that include what we’d normally charge for shipping?

- Client: Of course. It’s the total they’d pay us.
- You: That’s nice and simple for our customers to understand: I can see the attraction. But I can see some less
  scrupulous customers trying to game that system.
- Client: How so? You: Well, let’s say they buy a book for $25, and then select overnight shipping, the most expensive
  option. That’ll likely be about $30, making the
  whole order $55. We’d then make the shipping free, and they’d get overnight shipping on a $25 book for just $25._

**At this point the experienced developer stops. Deliver facts, and let the client make the decisions**

If it's not easy to get a feedback create a mockup and prototype to show to the client "is this way you meant?" this
gives you quick feedback loop.

**walk in your cline's shoes**
simple way to achieve this is by becoming a client - spend time on the clients daily work routine to understand better
the context.

### 46 Solving Impossible Puzzles

#### Tup 81 Don’t Think Outside the Box—Find the Box

Whenever you're solving a hard problem focus on constrains

You must challenge any preconceived notions and evaluate whether or not they are real, hard-and-fast constraints. It’s
not whether you think inside the box or outside the box. The problem lies in finding the box—identifying the real
constraints.

ask questions:

- Why are you solving this problem?
- What’s the benefit of solving it?
- Are the problems you’re having related to edge cases? Can you eliminate them?
- Is there a simpler, related problem you can solve?

When you have a problem with solving the problem always take a break and focus on something different.

### 47 Working Together

**Pair Programming** - one developer operates the keyboard, and the other does not. Both work on the problem together,
and can switch typing duties as needed.

**Mob Programming** - extension of pair programing but using more than 2 people (not necessarily ony developers).
you swap out the typist every 5-10 minutes.

#### Tip 82 Don’t Go into the Code Alone

### 48 The Essence of Agility

Manifesto values:

- **Individuals and interactions** over processes and tools
- **Working software** over comprehensive documentation
- **Customer collaboration** over contract negotiation
- **Responding to change** over following a plan

That is, while there is value in the items on the right, we value the items on the left more.

Agile is not about process but responding to change, to the unknowns after you set out.

The above values don't tell what to do. They tell you what to look for.

our recipe for working in an agile way:

1. Work out where you are.
1. Make the smallest meaningful step towards where you want to be.
1. Evaluate where you end up, and fix anything you broke.
1. Repeat

## 9 Pragmatic Projects

### 49 Pragmatic teams

team has to support the **no broken window** (those small imperfections that no one fixes) mentality.
Also be stayed alerted to avoid being **Boiled Frogs**. It’s even easier for teams as a whole to get boiled. People
assume that someone else is handling an issue, or that the team leader must have OK’d a change that your user is
requesting. Even the best-intentioned teams can be oblivious to significant changes in their projects.

**Communicate Team Presence** - team needs to communicate with the rest of the organization.
To outsiders, the worst project teams are those that appear sullen and reticent. They hold meetings with no structure,
where no one wants to talk. Their emails and project documents are a mess: no two look the same, and each uses different
terminology.

There is a simple marketing trick that helps teams communicate as one: generate a brand. When you start a project, come
up with a name for it, ideally something off-the-wall. (In the past, we’ve named projects after things such as killer
parrots that prey on sheep, optical illusions, gerbils, cartoon characters, and mythical cities.) Spend 30 minutes
coming up with a zany logo, and use it. Use your team’s name liberally when talking with people. It sounds silly, but it
gives your team an identity to build on, and the world something memorable to associate with your work.

Organize Fully Functional Teams - That means that you need all the skills to do that within the team: frontend, UI/UX,
server, DBA, QA, etc.
It allows to build end-to-end functionality in small pieces.

### 50 Coconut don't cut it

The goal of course isn’t to “do Scrum,” “do agile,” “do Lean,” or what-have-you. The goal is to be in a position to
deliver working software that gives the users some new capability at a moment’s notice. Not weeks, months, or years from
now, but now. For many teams and organizations, continuous delivery feels like a lofty, unattainable goal, especially if
you’re saddled with a process that restricts delivery to months, or even weeks. But as with any goal, the key is to keep
aiming in the right direction.

### 51 Pragmatic Starter Kit

covering three critical and interrelated topics:

- Version Control - is needed
- Regression Testing - Use all kinds of testing to make sure that your code is working properly.
  (Unit test, Integration tests, QA testing, performance testing) + make sure to test the state not the coverage
- Full Automation - if something can be automated do it (like setting up your environment, CI)

### 52 Delight Your Users

Understand the underlying expectations of value behind the project, you can start thinking about how you can deliver
against them:

- Make sure everyone on the team is totally clear about these expectations.
- When making decisions, think about which path forward moves closer to those expectations.
- Critically analyze the user requirements in light of the expectations. On many projects we’ve discovered that the
  stated “requirement” was in fact just a guess at what could be
  done by technology: it was actually an amateur implementation plan dressed up as a requirements document. Don’t be
  afraid to make suggestions that change the requirement if you can demonstrate that they will move the project closer
  to
  the objective.
- Continue to think about these expectations as you progress through the project.

### 53 Pride and Prejudice

You shouldn’t jealously defend your code against interlopers; by the same token, you should treat other people’s code
with respect. The Golden Rule (“Do unto others as you would have them do unto you’’) and a foundation of mutual respect
among the developers is critical to make this tip work.
We want to see pride of ownership. “I wrote this, and I stand behind my work.” Your signature should come to be
recognized as an indicator of quality. People should see your name on a piece of code and expect it to be solid, well
written, tested, and documented. A really professional job. Written by a professional.

## 10 Postface

Many nonembedded systems can also do both great good and great harm. Social media can promote peaceful revolution or
foment ugly hate. Big data can make shopping easier, and it can destroy any vestige of privacy you might think you have.
Banking systems make loan decisions that change people’s lives. And just about any system can be used to snoop on its
users. We’ve seen hints of the possibilities of a utopian future, and examples of unintended consequences leading to
nightmare dystopias. The difference between the two outcomes might be more subtle than you think. And it’s all in your
hands.

We have a duty to ask ourselves two questions about every piece of code we deliver: 
1. Have I protected the user? 
2. Would I use this myself?

