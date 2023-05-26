# Notes

Let's assume we have a class `A` which is responsible for sorting orders.
Class `A` has dependency on class `B` which is used in some stage on sorting algorithm.
We want to UniTest `A` class.

the structure looks something like this.

val orders = listOf<Order>()
val b = B()
val a = A(b)

a.sortOrders(orders)

fun sortOrders(orders: List<Order>) : List<Order> {
val doneOrders = order.filter { order -> b.isDone(order)
return doneOrders.sortByDate()
}

what is the proper approach to unit tests in the above case?
Should I mock the B class, if so, having a list of orders how can I assure that b.isDone() returns true for correct orders?
Do I need to specify the response for each order e.g.?

```kotlin
every {b.isDone(order1)} return true

every {b.isDone(order2)} return false
```
and so on for n orders?
