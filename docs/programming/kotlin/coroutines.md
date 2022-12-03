# Coroutines

are:
asynchronous - you have to wait for result
non-blocking - it doesn't block the thread
sequential code - no callback are required

`suspend` - key word for coroutines - waits until job will return, though different work can be processed on same thread
Coroutine needs:
- `Job` - cancellable object with lifecycle that completes
- `Dispatcher` - sends coroutines to different threads (like rx java) scheduleOn()
- `Scope` - combines information including Job and Dispatcher. Creation: `CoroutineScope(dispatcher + job)`

Job is just a representation of coroutine. There is no coroutine without job.

Coroutine scopes are just boxes in which coroutines run (wrapper around `CoroutineContext`).
It needs to have a root Job!
CoroutineContext is a `Set` of elements.
"most of the time coroutine scope and context are the same thing"

Dispatchers:

- `Main` - is adding a block to execute to a queue which means the block will be executed in future
- `Main.immediate` - if current thread is Main it will immediately execute code,
  otherwise it will fall back to `Main`
- `Default` - is using 2 or more threads
- `I.O.` is using 64 or more threads
- `Unconfinead` - is using the same thread as one which is started from

If dispatcher is not specify in scope nor as argument then `Dispatchers.Default` wil be used.
Dispatchers are using threads under the hood.

`withContext(Dispatchers.Default + NonCancellable) {...}`
adding `NonCancellable` guarantees that code inside will not be cancelled when parent coroutine will throw the exception

`CoroutineContext.ensureActive()` is checking if hob is Active otherwise throws cancellation exception
`CoroutineScope.coroutineContext.cancel(children)` allows to cancel all jobs inside coroutine scope