## Threading

In android each program (application) gets its own task.
This task is associated with isolated execution environment (sandbox).

```mermaid
flowchart LR
    A[PROCESS] <---> B["APPLICATION + SANDBOX"]
```

Each process can host one or more tasks. In Android these are Threads.
Threads _share_ the execution environment with the parent process. They can communicate and exchange data.


**thread** - single in process task

creation:
1. extend thread class
2. creating runnable and passing to constructor of `Thread(myRunnable)`

advantage of 2nd approach is using "composition over inheritance"

Android UI thread lives as long as app

Memory assignment

```kotlin
class MyActivity: Activity {
    lateinit var myRepo: Repo
    override fun onCreate() {
        myRepo = Repo()
    }
}
```

Memory model of the above code
```mermaid
flowchart TB
    MyActivity --Reference--> MyRepo
```

**Garbage Collector, GC** - system process which automatically reclaims memory 
by discarding objects that are no longer in use (not reachable) 

Object reachability flow:

```mermaid
flowchart LR

    A{"is root?"} --yes--> B[reachable]
    C --yes--> D{"is any parent reachable?"}
    D --yes--> B
    A --no--> C{is referenced?}
    C --no--> E[not reachable]
    D --no--> E
    
    style B fill:#0F0
    style E fill:#F00
    
```

### Multithreading

decomposition of app logic into multiple concurrent tasks


