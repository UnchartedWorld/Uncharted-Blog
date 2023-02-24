---
title: "Third Week"
#description: <descriptive text here>
date: 2023-02-20T17:40:57Z
draft: true
toc: true
image: ""
tags: []
series: "Functional Programming"
---

# Week 3 – Lists I

## What is a linked list?

A [linked list](https://en.wikipedia.org/wiki/Linked_list) is a linear data structure where each node stores a value and each node points to its successor node. 5→4 and so on. In Scala, this can look like:

```Scala
scala> 5 :: 4 :: 3 :: 2 :: 1 :: Nil
val res0: List[Int] = List(5, 4, 3, 2, 1)
```

Remember Big-O notation? That stuff is important, at least **at scale**, so considering operation timing is important. So, for the sake of quickly running through it, I'll list below Linked List operations and their Big-O timing:

- Accessing the **head** of a list is O(1)
	- `xs.head`
- Accessing the **last** (or frankly, any element other than the 1st) is O(n)
	- `xs.last`
- Adding an element to the **front** of the list is O(1)
	- `val ys = 6 :: xs`
	- Alternatively, `val zs = 7 :: xs.tail`

You may want to know that this makes use of **structure sharing**. This optimises memory space usage because even when creating 3 lists, it uses the same nodes, i.e. `ys` uses the same node as `xs` and so on. These are also persistent, something that allows us to retain the original values.

However, there is one slight problem. When writing Functional code, you really want to **avoid mutable variables and data structures**. This is because when they're mutable, the original values can change. This isn't exactly something we want – since it means we lose that aforementioned structure sharing, and as such you should endeavour to avoid this.

## Scala List

Scala (as it should) makes this easier on us, providing the immutable.List collection. Normally I'd love to bore you with discussions of UML, but I'll go through this quickly.

A List type is a *generic* [A] (meaning it can have many types) and a *covariant* [+A]. A list can either be *Nil* or can be constructed into, well, a list. You've likely asked, “what is a covariant?”. Let's have two fake Lists for the sake of explaining – in part thanks to a helpful [StackOverflow user](https://stackoverflow.com/a/41586692). 

Say we have `List[A]` and `List[B]`. If `B` extends `A` and the passing of `List[B]` is valid, then it is considered Covariant. Below is a diagram from [Kamil Korzekwa's blog](http://blog.kamkor.me/Covariance-And-Contravariance-In-Scala/) that may help:

![Diagram of covariance using the analogy of drinks and sub-drinks](http://blog.kamkor.me/images/Covariance-And-Contravariance-In-Scala/drinks_model.png)

In his example, an employer wants a soft drink vending machine. So, rather than have a Coca-Cola machine, an orange juice machine and so on, you make a vending machine class and the others inherit it and are valid because they're all the same base type – Drink.

A good example of code that *isn't* covariant is seen below:

```Scala
class A
case class B(i: Int) extends A

val bs = new java.util.ArrayList[B]()
bs.add(B(1))

def count(xs:java.util.ArrayList[A]): Int = xs.size()
```

This will fail because it's a type mismatch. Count requires an ArrayList[A], but you've provided it an ArrayList[B]. Whilst they both use the same data type (Int), because they're technically different types of Lists, it's not valid.

So, how do we fix this? Simple really:

```Scala
class A
case class B(i: Int) extends A

def len(xs:List[A]): Int = xs.length

scala> len(List(B(1)))
// Outputted in val xy:Int, this will = 1.
```

This works because the majority of Scala's collection classes are covariant in a generic type parameter.

### Constructing Scala lists

You would've noticed at the beginning of this, we used `::`. This actually syntactic sugar – something we covered in the [previous week here]({{< ref "/Second-Week#Syntactic" >}} "Second Week").