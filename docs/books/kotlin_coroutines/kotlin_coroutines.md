# Kotlin Coroutines Deep Dive

## Part 1: Understanding KotlinCoroutines

### Why Kotlin Coroutines?

Threads issues:

- There is no mechanism here to cancel these threads, so we
  often face memory leaks.
- Making so many threads is costly.
- Frequently switching threads is confusing and hard to manage.
- The code will unnecessarily get bigger and more complicated.

Callbacks:

- Getting data parallelized, is not so straightforward with callbacks
- supporting cancellation requires a lot of additional effort.
- The increasing number of indentations make this code hard to
  read (code with multiple callbacks is often considered highly
  unreadable). Such a situation is called “callback hell”.
- When we use callbacks, it is hard to control when things are triggered.

Rxjava/ReactiveStreams

- You need to learn different functions, like subscribeOn, observeOn, map, or subscribe,
- Cancelling needs to be explicit.
- Functions need to return objects wrapped inside Observable or Single classes.
- In practice, when we introduce RxJava, we need to reorganize our code a lot.

## Using Kotlin coroutines

core functionality is the ability to **suspend** a coroutine at some point and resume it in the future.
Suspended coroutine released the thread, so thread is not blocked!

`suspendCoroutine<Unit> {continuation ->  continuation.resume(Unit}`

suspendCoroutine needs to be called with resume to progress, otherwise it will always be suspended.

**Suspending a coroutine, not a function**

we suspend a coroutine, not a function. Suspending functions are not coroutines, just
functions that can suspend a coroutine.

## Coroutines under the hood

- suspend fun in reality are fun with additionally parameter at the end: continuation
- Suspending functions are like state machines, with a possible state at the beginning of the function and after each
  suspending
  function call.
- Both the number identifying the state and the local data are kept in the continuation object.
- Continuation of a function decorates a continuation of its caller function; as a result, all these continuations
  represent a call stack that is used when we resume or a resumed function completes.

## Part 2: Kotlin Coroutines library

### Coroutine builders

Suspend function can't be called from normal function, it can either be started in another suspended function or from
coroutine builder - bridge from the normal to the suspending world.

most common used:

- launch
- runBlocking
- async

**launch**
The way `launch` works is conceptually similar to starting a new `thread` (thread function)
`launch` is an extension function on the `CoroutineScope` interface. This is part of an important mechanism called
**structured concurrency**, whose purpose is to build a relationship between the parent coroutine and a child coroutine.
To some degree, how `launch` works is similar to a `daemon thread` but much cheaper.

```kotlin
fun main() {
    thread(isDaemon = true) {
        Thread.sleep(1000L)
        println("World!")
    }
    thread(isDaemon = true) {
        Thread.sleep(1000L) println ("World!")
    }
    thread(isDaemon = true) {
        Thread.sleep(1000L)
        println("World!")
    }
    println("Hello,")
    Thread.sleep(2000L)
}
```

```kotlin
fun main() {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    Thread.sleep(2000L)
}
```

**runBlocking builder** (currently rarely used)
`runBlocking` is a very atypical builder. It blocks the thread it has been started on whenever its `coroutine` is
suspended (similar to suspending main).
This means that `delay(1000L)` inside runBlocking will behave like `Thread.sleep(1000L)`:

```kotlin
fun main() {
    Thread.sleep(1000L)
    println("World!")
    Thread.sleep(1000L)
    println("World!")
    Thread.sleep(1000L)
    println("World!")
    println("Hello,")
}
```

```kotlin
fun main() {
    runBlocking {
        delay(1000L)
        println("World!")
    }
    runBlocking {
        delay(1000L)
        println("World!")
    }
    runBlocking {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

Use cases in which `runBlocking` is used:

- main function, where we need to block the thread, because otherwise the program will end.
- unit tests, where we need to block the thread for the same reason.

**async builder**
Similar to `launch`, but produce a value.
The `async` function returns an object of type `Deferred<T>`, where `T` is the type of the produced value.
`Deferred` has a suspending method await, which returns this value once it is ready.
Just like the `launch` builder, `async` starts a coroutine immediately when it is called. So, it is a way to start a few
processes at once and then `await` all their results. The returned `Deferred` stores a value inside itself once it is
produced, so once it is ready it will be immediately returned from `await`. However, if we call `await` before the value
is
produced, we are suspended until the value is ready.

```kotlin
fun main() = runBlocking {
    val res1 = GlobalScope.async {
        delay(1000L)
        "Text 1"
    }
    val res2 = GlobalScope.async {
        delay(3000L)
        "Text 2"
    }
    val res3 = GlobalScope.async {
        delay(2000L)
        "Text 3"
    }
    println(res1.await())
    println(res2.await())
    println(res3.await())
}
```

How the `async` builder works is very similar to `launch`, but it has additional support for returning a value. If all
`launch` functions were replaced with `async`, the code would still work fine. But don’t do that! `async` is about
producing a
value, so if we don’t need a value, we should use `launch`.

```kotlin
fun main() = runBlocking {
// Don't do that!
// this is misleading to use async as launch 
    GlobalScope.async {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    delay(2000L)
}
```

The async builder is often used to parallelize two processes, such as obtaining data from two different places, to
combine them together.

```kotlin
scope.launch {
    val news = async {
        newsRepo.getNews()
            .sortedByDescending { it.date }
    }
    val newsSummary = newsRepo.getNewsSummary() // we could wrap it with async as well,
// but it would be redundant
    view.showNews(
        newsSummary,
        news.await()
    )
}
```

### Structured Concurrency

If a coroutine is started on `GlobalScope`, the program will not wait for it. As previously mentioned, coroutines do not
block any threads, and nothing prevents the program from ending. This is why, in the below example, an additional delay
at the end of `runBlocking` needs to be called if we want to see “World!” printed.

```kotlin
fun main() = runBlocking {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    GlobalScope.launch {
        delay(2000L)
        println("World!")
    }
    println("Hello,")
    //    delay(3000L)
}

// Hello,
```

Why do we need this `GlobalScope` in the first place?
It is because launch and async are extension functions on `CoroutineScope`.
However, if you take a look at the definitions of these and of `runBlocking`, you will see that the
block parameter is a function type whose receiver type is also `CoroutineScope`.
This means we don't need `GlobalScope`, instead, `launch` can be called on the receiver provided by
`runBlocking`.

```kotlin
fun main() = runBlocking {
    this.launch { // same as just launch
        delay(1000L)
        println("World!")
    }
    launch { // same as this.launch
        delay(2000L)
        println("World!")
    }
    println("Hello,")
}
```

A parent provides a scope for its children, and they are called in this scope.
This builds a relationship that is called a structured concurrency.
Here are the most important effects of the parent-child relationship:
• children inherit context from their parent (but they can also overwrite)
• a parent suspends until all the children are finished
• when the parent is cancelled,its child coroutines are cancelled too
• when a child raises an error,it destroys the parent as well

Notice that, unlike other coroutine builders, runBlocking is not an extension function on CoroutineScope. This means
that it cannot be a child: it can only be used as a root coroutine (the parent of all the children in a hierarchy).
This means that runBlocking will be used in different cases than other coroutines. As we mentioned before, this is very
different from other builders.

### Coroutine context

CoroutineContext is conceptually similar to a map or a set collection. It is an indexed set of Element instances, where
each Element is also a CoroutineContext. Every element in it has a unique Key that is used to identify it. This way,
CoroutineContext is just a universal way to group and pass objects to coroutines. These objects are kept by the
coroutines and can determine how these coroutines should be running (what their state is, in which thread, etc).

Adding contexts - What makes CoroutineContext truly useful is the ability to merge two of them together.
When two elements with different keys are added, the resulting context responds to both keys.

```kotlin

fun main() {
    val ctx1: CoroutineContext =
        CoroutineName("Name1") println (ctx1[CoroutineName]?.name) // Name1 println(ctx1[Job]?.isActive) // null
    val ctx2: CoroutineContext =
        Job() println (ctx2[CoroutineName]?.name) // null println(ctx2[Job]?.isActive) // true, because "Active" // is the default state of a job created this way
    val ctx3 = ctx1 + ctx2 println (ctx3[CoroutineName]?.name) // Name1 println(ctx3[Job]?.isActive) // true
}
```

When another element with the same key is added, just like in a map, the new element replaces the previous one.

```kotlin
fun main() {
    val ctx1: CoroutineContext = CoroutineName("Name1") println (ctx1[CoroutineName]?.name) // Name1
    val ctx2: CoroutineContext = CoroutineName("Name2") println (ctx2[CoroutineName]?.name) // Name2
    val ctx3 = ctx1 + ctx2
    println(ctx3[CoroutineName]?.name) // Name2
}
```

Subtracting elements - Elements can also be removed from a context by their key using the minusKey function.

Folding context - If we need to do something for each element in a context, we can use the fold method, which is similar
to fold for other collections. It takes:

**Coroutine context and builders**
So CoroutineContext is just a way to hold and pass data. By default, the parent passes its context to the child, which
is one of the parent- child relationship effects. We say that the child inherits context from its parent.

Since new elements always replace old ones with the same key, the child context always overrides elements with the same
key from the parent context. The defaults are used only for keys that are not specified anywhere else. Currently, the
defaults only set Dispatchers.Default when no ContinuationInterceptor is set, and they only set CoroutineId when the
application is in debug mode.
There is a special context called Job, which is mutable and is used to communicate between a coroutine’s child and its
parent.

**Accessing context in a suspending function**
Context is referenced by continuations, which are passed to each suspending function. So, it is possible to access a
parent’s context in a suspending function. To do this, we use the coroutineContext property, which is available in every
suspending scope.

```kotlin

suspend fun printName() {
    println(coroutineContext[CoroutineName]?.name)
}

suspend fun main() = withContext(CoroutineName("Outer")) {
    printName() // Outer
    launch(CoroutineName("Inner")) {
        printName() // Inner
    }
    delay(10)
    printName() // Outer
}
```

**Creating our own context**
It is not a common need, but we can create our own coroutine context pretty easily. To do this, the easiest way is to
create a class that implements the CoroutineContext.Element interface. Such a class needs a property key of type
CoroutineContext.Key<*>. This key will be used as the key that identifies this context. The common practice is to use
this class’s companion object as a key. This is how a very simple coroutine context can be implemented:

```kotlin
class MyCustomContext : CoroutineContext.Element {
    override val key: CoroutineContext.Key<*> = Key

    companion object Key : CoroutineContext.Key<MyCustomContext>
}
```

### job

Conceptually, a job represents a cancellable thing with a lifecycle. Formally, Job is an interface, but it has a
concrete contract and state, so it might be treated similarly to an abstract class.

A job lifecycle is represented by its state. Here is a graph of states and the transitions between them:
![jobs_lifecycle.png](jobs_lifecycle.png)

In the “Active” state, a job is running and doing its job. If the job is created with a coroutine builder, this is the
state where the body of this coroutine will be executed. In this state, we can start child coroutines. Most coroutines
will start in the “Active” state. Only those that are started lazily will start with the “New” state. These need to be
started in order for them to move to the “Active” state. When a coroutine is executing its body, it is surely in the
“Active” state. When it is done, its state changes to “Completing”, where it waits for its children. Once all its
children are done, the job changes its state to “Completed”, which is a terminal one. Alternatively, if a job cancels or
fails when running (in the “Active” or “Completing” state), its state will change to “Cancelling”. In this state, we
have the last chance to do some clean-up, like closing connections or freeing resources (we will see how to do this in
the next chapter). Once this is done, the job will
move to the “Cancelled” state.

Every coroutine builder from the Kotlin Coroutines library creates its own job. Most coroutine builders return their
jobs, so it can be used elsewhere. This is clearly visible for launch, where Job is an explicit result type.

There is a very important rule: Job is the only coroutine context that is not inherited by a coroutine from a coroutine.
Every coroutine creates its own Job, and the job from an argument or parent coroutine is used as a parent of this new
job.There is a very important rule: Job is the only coroutine context that is not inherited by a coroutine from a
coroutine. Every coroutine creates its own Job, and the job from an argument or parent coroutine is used as a parent
of this new job.
The parent can reference all its children, and the children can refer to the parent. This parent-child relationship (Job
reference storing) enables the implementation of cancellation and exception handling inside a coroutine’s scope.

Structured concurrency mechanisms will not work if a new Job context replaces the one from the parent. To see this, we
might use the Job() function, which creates a Job context (this will be explained later).

```kotlin
fun main(): Unit = runBlocking {
    launch(Job()) { // the new job replaces one from parent
        delay(1000)
        println("Will not be printed")
    }
}
// (prints nothing, finishes immediately)
```

n the above example, the parent does not wait for its children because it has no relation with them. This is because the
child uses the job from the argument as a parent, so it has no relation to the runBlocking.
When a coroutine has its own (independent) job, it has nearly no relation to its parent. It only inherits other
contexts, but other results of the parent-child relationship will not apply. This causes us to lose structured
concurrency, which is a problematic situation that should be avoided.

The first important advantage of a job is that it can be used to wait until the coroutine is completed. For that, we use
the join method. This is a suspending function that suspends until a concrete job reaches a final state (either
Completed or Cancelled).

```kotlin
fun main(): Unit = runBlocking {
    val job1 = launch {
        delay(1000)
        println("Test1")
    }
    val job2 = launch {
        delay(2000)
        println("Test2")
    }
    job1.join()
    job2.join()
    println("All tests are done")
}
// (1 sec)
// Test1
// (1 sec)
// Test2
// All tests are done
```

The Job interface also exposes a children property that lets us reference all its children. We might as well use it to
wait until all children are in a final state.

```kotlin
val children = coroutineContext[Job]?.children
children?.forEach { it.join() }
println("All tests are done")
```

### Cancellation

Basic cancellation
The Job interface has a cancel method, which allows its cancellation. Calling it triggers the following effects:
• Such a coroutine ends the job at the first suspension point (delay in the example below).
• If a job has some children, they are also cancelled (but its parent is not affected).
• Once a job is cancelled, it cannot be used as a parent for any new coroutines. It is first in the “Cancelling” and
then in the
“Cancelled” state.

We might cancel with a different exception (by passing an exception as an argument to the cancel function) to specify
the cause. This cause needs to be a subtype of CancellationException, because only an exception of this type can be used
to cancel a coroutine.
After cancel, we often also add join to wait for the cancellation to finish before we can proceed. Without this, we
would have some race conditions. The snippet below shows an example in which without join we will see “Printing 4” after
“Cancelled successfully”.

```kotlin
suspend fun main() = coroutineScope {
    val job = launch {
        repeat(1_000) { i ->
            delay(100)
            Thread.sleep(100) // We simulate long operation
            println("Printing $i")
        }
    }
    delay(1000)
    job.cancel()
    println("Cancelled successfully")
}
// Printing 0
// Printing 1
// Printing 2
// Printing 3
// Cancelled successfully
// Printing 4
```

Adding `job.join()` would change this because it suspends until a coroutine has finished cancellation (I guess currently
it's in cancelling state, hence it's still able to produce values).
To make it easier to call cancel and join together, the kotlinx.coroutines library offers a convenient extension
function with a self-descriptive name, `cancelAndJoin`.