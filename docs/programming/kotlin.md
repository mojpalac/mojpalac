# Kotlin

kotlin has property: List.indices which returns index of last element


``` java
 <out MutableList<X>> //  means that we can't add anything to this list, we can only read valuse from it

```

### delegates properties:

``` java
by Lazy<T>
by Delegates.observable(){} // will call specific block of function when the observed value has changed
by Delagates.vetoable() // will call specific block which returns boolean -> true assing new value
by Delegates.notNull() // will throw IllegalStateException if not initilized"
```
by Lazy allows to also to initialize values, which can allow to omit NPE in case when fe. object is inside sealed class that takes value like: object: Tree: Plant("green")


### coroutines

are:
asynchronous - musisz czekac na rezultat
non-blocking - nie blokuje UI threada
sequentional code - nie musisz miec callbackow

suspend - key word for coroutines - czeka az coroutine zwroci wynik, ale inna praca moze sie wykonywac
Coroutine needs:
- Job - cancellable object with lifecycle that complets
- Dispatcher - sends coroutines to different threads (like rx java) scheduleOn()
- Scope - combines informations including Job and Dispatcher - tworzenie CoroutineScope(Dispatcher + job)