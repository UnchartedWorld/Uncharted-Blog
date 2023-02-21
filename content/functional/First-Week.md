---
title: "First Week" 
date: 2023-02-20T16:11:12Z
draft: false
toc: true
image: ""
tags: []
series: "Functional Programming"
type: "Functional"
---

# First Week - Introduction 

## Background

Functional Programming (FP) is one of the major paradigms that can be utilised to create a variety of applications and software solutions, alongside: 
- [Imperative](https://www.ionos.com/digitalguide/websites/web-development/imperative-programming/) programming, such as C, Fortran and so on. A good example from the provided link is shown below (PHP specifically) in case you don't understand it.

```php
$participantlist = [1 => 'Peter', 2 => 'Henry', 3 => 'Sarah'];
$firstnames= [];
foreach ($participantlist as $id => $name) {
    $firstnames[] = $name;
}
```
- Conversely, [Object-Oriented](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/oop) programming is (in?)famous for its usage of classes to adhere to the 4 basic principles of OOP, which are: 
	- *Abstraction, encapsulation, inheritance & polymorphism*
- Below is a very small example [from Caio Cesar](https://dev.to/caiocesar/c-object-oriented-programming-oop-cheat-sheet-5bkf) of OOP in C#:

```C#
public class Book
{
        string Author { get; set; }

        string Name { get; set; }

        public double GetPrice()
        {
            return 100.00;
        }
}
```
This understandably doesn't demonstrate the full 4 principles, but the concept remains the same. 

Functional Programming is a style that emerged in the 1950s through [LISP](https://en.wikipedia.org/wiki/Functional_programming#History), a high-level programming language. This was based on a theory of functions, referred to as the *Lambda Calculus*, introduced in no small part by **Alonzo Church** in the 1930s. 

Do you need to know all of this to understand Functional Programming? No, but it helps nonetheless.

## So, why are we not using Imperative/Object-Oriented Programming?

In Imperative programming, the ordering of statements proves immensely crucial in how a given program will be run. For example, say we had the following code: 

```python
t = 5
x = 6

t = x // T will = 6
```
We know that t  = 6 at the end, but what happens if we switch it around, i.e.

```python
t = 5
x = 6

x = t // X will = 5
```
This may not be a problem for some circumstances, but it shows that the sequence of declaration is pivotal. However, generally you will need to assign variables before usage in a for-loop, for example.

### The problem with imperative programming & variables

A great way to demonstrate the issues with imperative programming is to consider concurrency. When a variable is needed at the same time, how do you handle that? Normally, this is done via locks to manage what has access to a given variable at specific times. However, this is complicated to manage and can scale rather poorly - something [Peyton Jones](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/beautiful.pdf) noted in his article, *Beautiful Concurrency*.

Furthermore, it is entirely possible to accidentally update the value of a variable without intention. This can occur when one has a shared point pointer that needs to be accessed by two different things at the same time.

```Java
class Count() {
  private int x = 0;
  public int get() {return x;}
  public void inc() {x=x+1;}
  public String toString() { return “” + x;}
}

class Event() {
  private Count c;
  public Event(Count k) {c = k;}
  public void next() {c.inc();}
  public Count getC() {return c;}
  //…
}

Count zero = new Count;

Event e1 = new Event(zero);
Event e2 = new Event(zero);

println(e1.getC());
e1.next(); 
println(e2.getC()); // e2 CAN become the same as e1, despite technically not being the same Event variable.

```


There are ways to handle this, but it can cause issues even after resolving it. Finally, sometimes Imperative/Object Oriented Programming can just be difficult to read. __You could just write better code__ and avoid complex declarations, but sometimes that is impossible. A great example is shown below.


```C
int c = 3;
int d = 3;
int x = c++ * --d;				// x = 6;  c = 4;  d = 2
int y = (x = x-1) * ((x = x+1) + 1);
// is it	= 5 * 7
// or	    = 6 * 8
// What about
x[i] = i++; 
```


So, we can use Functional Programming to **remove assignments entirely**, and this intentional lack of assignments helps us reduce the chances of race conditions, order of statements or anything else like that.

The kind of things we can have in Functional Programming and more specifically Scala are shown below:


```Scala
// We have definitions
val x: Int = 5
val y: Int = x
// We have expressions
println(x*2 + (y*x))
// We have methods that look like functions
def inc(x: Int) = x+1
// And we have functions
val dbl = (x: Int) => x*2
```

I should note that the existence of methods *technically* violates the whole functional paradigm

## Functional Programming - An Introduction

**Lambda calculus** defines the logic that Functional Programming is based on. Underneath all the (frankly, more legible) syntax of - say - Scala, a standard function looks like this: 

Syntax:	expr	::=	var 
 			 |	expr  expr
				|	“(λ”  var  “->”  expr  “)”
				|	“(“  expr  “)”

To explain, *expr expr* is an assignment and the λ (lambda) is an abstraction. This is what most functions are eventually simplified down to, so knowing this isn't entirely necessary for the purposes of programming.

You can name functions in Scala - useful if you plan on re-using it. Some examples of that are shown below:
```Scala
val incr: Int => Int = n => n+1
val isPalindrome: String => Boolean = s => s == s.reverse
val double: Float => Float = x => x*2.0
```

### Anonymous Functions
**Anonymous Functions** are the kinda thing you've used if you ever did a .map in most language. For example, (λ n -> n + 1) can be explained as "take the value n and return n + 1". Put 3 outside (which, in this case, would serve as *n*), and you've really just wrote 3 + 1, which = 4. 

In Scala, this would look like `(n: Int) => n+1`, and applied like `((n: Int) => n+1)(3)`.  Functions *don't need to be declared prior to use*. In theory, you can declare them AS you use them i.e. `List(1, 2, 3, 4, 5).map( n => n * 2 )`.

### Curried Functions

**Curried functions** are functions that take more than one argument (technically false, it's one input, instead you return a function, which can have it's own singular argument). Here's an example:

```Scala
val add: Int => Int => Int = x => y => x+y
// Parenthesised, it's
val add: Int => (Int => Int) = x => (y => x+y)
```
In this, Add takes an Int argument as input, returning a function. In this case, x is the input. 

### Partial Application

**Partial application** is where one uses a function with potentially two arguments, but you only provide one. By doing this, you can return just the function itself. Additionally, you can construct new functions by partially applying existing ones.

For example, let's use the prior *add* function. If we did `add(2)`, this would be a function that adds two. You could go on further to create functions based on prior functions, thus showing the power that partial application may have. One method of doing so is **composition** and **andThen**.

### Composition

**Composition**, as stated before, allows you to utilise multiple different functions together to reach some sort of end-goal. Let's pretend we have two functions, such as `def dbl(x: Int) = x * 2`, and `def decr(x: Int): Int = x - 1`. If I wanted to use both of these, how would I do that? One example is: 
```Scala
val nums: List[Int] = (1 to 10).toList

@main def fakeMain(): Unit = 
	println(nums.map(dbl.compose(decr))) // This would be List(0, 2, 4, 6, 8, 10, 12, 14, 16, 18), as we do decr first, then dbl.
	println(nums.map(decr.compose(dbl))) // This would be List(1, 3, 5, 7, 9, 11, 13, 15, 17, 19), as we do dbl first, then decr.
```
Simply put, with compose, you do the latter before the former. We could pipeline compose multiple times, and this still remains consistent. 

What if I wanted to do dbl first, then decr, but didn't want to change the order (either because we're particular about it, or it makes more sense)? We can do that using **andThen**, and it would look something like:
```Scala
val nums: List[Int] = (1 to 10).toList

@main def fakeMain(): Unit = 
	println(nums.map(dbl.andThen(decr))) // This would be List(1, 3, 5, 7, 9, 11, 13, 15, 17, 19), as we did dbl, and then decr.
	println(nums.map(decr.andThen(dbl))) // // This would be This would be List(0, 2, 4, 6, 8, 10, 12, 14, 16, 18), as we do decr first, then dbl.
```
This is called **forward composition** - forward because it goes left to right i.e. dbl, then decr in the first println.

It should be noted that functions that:
- Take functions as parameters and/or
- Return functions as results
- Are generally called *higher order functions*.

### Object-Oriented Notation vs Infix Notation

Object oriented (OO) notation is rather simple. If we had a method that we created, we can do something like *receiver.method*. This works wonderfully for OOP, but it can look weird with Functional Programming i.e. 2.+3. So, we can write it in **infix notation**, which can look like 2 + 3. Another example is `List(1,2,3).map(inc)` = `List(1,2,3) map inc`. Whichever one of these you choose - OO notation or Infix respectively, it should be what you feel is easier to read and understand.

### Placeholder syntax

Sometimes we may need some form of placeholder syntax to stand in place of a parameter. That's where the underscore shines. Say we have the following code: `List(1,2,3) map (_+1)`. The underscore can be inferred to mean n, x or whatever value is used in the function. In this case, it would be x + 1. If the values can be easily inferred, placeholder syntax can prove powerful.

### Curried methods
These are methods which can take multiple parameter lists. In essence, you can make a function out of singular arguments put together to form a full function.
```Scala
def pre(c: Char)(s: String): String = c +: s
def post(c: Char)(s: String): String = s :+ c

List("Scala", "is", "fun") map pre(‘<‘) map post(‘>’)
// = List (“<Scala", “<is", “<fun") map post(‘>’)
// = List(“<Scala>”, “<is>”, “<fun>”)
```
In the code above, pre takes a Character value and a String and adds it as required, similar to post. This is then used to modify each value in the list to form a new list with each element encased in <>.