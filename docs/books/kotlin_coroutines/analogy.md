# kotlin coroutines analogy

## CoroutineContext, CoroutineScope, Coroutine, Jobs

Imagine you are the manager(CoroutineContext) of a restaurant(CoroutineScope). As the manager, you provide a specific
set
of rules and resources for your employees(coroutines) to work within the restaurant. These rules and resources
collectively represent the CoroutineContext. For example, the restaurant's working hours, uniforms, cooking equipment,
and the specific tasks employees can perform (like cooking, serving, cleaning) are part of the CoroutineContext.

Every employee (coroutine) you hire in your restaurant is given a specific job assignment (analogous to a Job). Each job
has a set of responsibilities and tasks that the employee needs to perform. For instance, the chef's job is to cook
meals, the server's job is to deliver food to customers, and the cleaner's job is to keep the restaurant tidy.

Now, let's see how they relate to each other and what happens when we cancel one Job:

## Cancel all coroutines

When you, as the manager, decide to close the restaurant (call cancel() on the CoroutineScope), here's what happens:

    All the employees (coroutines) working in the restaurant (CoroutineScope) will be notified that the restaurant is closing (cancellation is requested).

    Each employee (coroutine) checks their job assignment (Job) and sees the notification of restaurant closure.

    The employees (coroutines) will stop working on their current tasks, clean up their workstations, and leave the restaurant.

## Coroutine cancels itself

Imagine, Each employee has a specific task (job) they are responsible for.

    Chef (Coroutine A) - Responsible for cooking dishes.
    Server (Coroutine B) - Responsible for serving food to customers.

Now, let's say the Chef (Coroutine A) is cooking a dish in the kitchen when they notice a fire has broken out in the
restaurant.
Realizing the danger, the Chef immediately informs the manager (CoroutineContext), about the fire (cancellation propagates to the CoroutineContext).
Manager in turn cancel all other employees' tasks (coroutines) within the restaurant (CoroutineScope).