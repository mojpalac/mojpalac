# Princeton university algorithms part 1

## 1.5 Union Find

**Dynamic connectivity**

Given a set of N objects:

- Union command: connect two objects (commonly known as p and q)
- Find/connected query: is there a patch connecting the two objects?

e.g. is there a connectivity between 0 and 6?

```
union(1, 2)
union(3, 4)
union(5, 6)
connected(0,6) - NO
union(7, 8)
union(7, 9)
union(2, 8)
union(0, 5)
connected(0,6) - YES
union(1, 9)
```

``` mermaid
flowchart LR
  subgraph connected component
  direction LR
    5 --> 6
    0 --> 5
  end
      3 --> 4
      1 --> 2
      7 --> 8
      7 --> 9
      2 --> 8
      1 --> 9
```

Connected components - maximal set of object that are mutually connected

it the above examples we have 3 components: `{ 0 }` `{ 1 4 5 }` `{ 2 3 6 7 }`

### Implementations

#### Quick Find (eager algorithm)

given an array:
[0,1,1,8,8,0,0,1,8,8] - known as id[]

we connect pairs by using:
- index as `p`
- value as `q` 

in the above example it gives as below connected components

``` mermaid
flowchart LR
0 --> 0
1 --> 1
2 --> 1
3 --> 8
4 --> 8
5 --> 0
6 --> 0
7 --> 1
8 --> 8
9 --> 8
```

To merge components containing `p` and `q` change **all** entire whose id equals id[p to id[q]]
In above case, if we are doing a union of `6` and `1` then all **values** for component {0, 5, 6 } needs to change id to used in the second component - which is 1.
array after change
[**1**,1,1,8,8,**1**,**1**,1,8,8] - known as id[]

The change is always based on order of the IDs - we change the first one to the **second** one


the above is kind of an issue when we have huge array.