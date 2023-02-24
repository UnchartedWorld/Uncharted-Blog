---
title: Second Week
#description: <descriptive text here>
date: 2023-02-20T16:17:06Z
draft: false
toc: true
image: ""
tags: []
series: Functional Programming
type: Functional
---

# Second Week – Recursion

## Method calling in Scala – What happens?

Let us take the following mock code in Scala: 

```Scala
def m3(z: Int): Int =
  val zz = z * z
  zz + z // 650
  
def m2(y: Int): Int =
  val yy = y*y
  yy + m3(yy) // 675
  
def m1(x: Int): Int =
  x * m2(x) // 3375
  
def main(…) …

val a = m1(5) + 1 // 3376
```

`val a` makes a method call to the function m1, so at the moment it equals nothing. However, m1 makes a call to m2 and m2 makes a call to m3. Each internal `val`'s value *does* change as it makes it each call towards m3, such as m1's *x* being 5, m2's `val yy` being 25 (as y = x, which is 5 as we know). You can probably infer what m3's *z*, and thus *zz* equals at that point. Then it'll go back down through each function towards m1, as shown by the commented values. 

This works splendidly, but you can likely infer that this isn't exactly pleasant and succinct code to read. If you're trying to write functional code, you probably don't want the headache of *long* functional code, amongst other reasons. Thus, **recursion** is your best friend.

## Recursive Methods – How they look and stack frame consideration

Below is an example of a recursive function in Scala. This is essentially the exact same as the method in the last section, but made recursive.
```Scala
def r(n: Int): Int =
  if n==0 then
    0
  else
    1 + r(n-1)			// 3

def main(…) …
  val a = r(3) + 1      // 3 + 1
```

Recursive calls that happen to require local storage will need to use up [stack frames](https://fpsimplified.com/scala-fp-Jvm-Stacks-Frames-2.html). This is, unsurprisingly, a large problem. Despite advances in hardware, stack overflows (not the website) can and will occur with recursive functions *if* they receive far too many calls. For example, `def add(k: Int, N: Int): Int = if k <= N then k + add(k+1, N) else 0` with the input `add(1,4)` would require *4* calls, which all need to retain the relevant local values and return addresses. How do we even avoid this? This is where the implementation of **good tail recursion** will come to aid you.

### Loops – A quick note

Just worth noting, loops are a little different in Scala. Remember, we **don't have access to assignments**. This makes the standard method of loops difficult to directly recreate. This isn't a huge problem, as Scala has its own way to loop.

```Scala
var s = 0
var k = 1

while k <= N do
    s = s + k
    k = k + 1
```
As indicated, this works, but has a few issues. Both *s* and *k* are mutable, which is something we want to avoid in functional programming if we can. So, as always, optimisation can be made.

```Scala
@tailrec
def sum(s: Int, k: Int, N: Int): Int = if k <= N then sum(s+k, k+1, N) else s
```
This is more performant as the bytecode for this function can be assigned to a single stack frame, resolving the issue we noted previously. [Alvin Alexander](https://alvinalexander.com/scala/fp-book/tail-recursive-algorithms/) made a rather helpful guide on how to write tail recursive functions.


## Tail Recursion

Tail recursive functions can be implemented by the compiler *as efficiently* as a loop. It should be noted that the recursive call is the last thing a function does, with no need to return to the prior calls to calculate the result. This is because the information is carried around in *parameter lists*. However, there is one key thing you must consider when making tail recursive functions. The recursive call **must be the last thing a function does**. If not, it's not a tail recursive function. This will result in it not using a single stack trace & we've already established why that's bad. For example, the following code is **not** tail recursive:

```Scala
def fact(n: Long): Long = if n==0 then 1 else n * fact(n-1)
```
What's wrong with this? The fact it ends with `n * fact(n-1)`. The `n *` is the problem part. The multiplication operator causes a separate stack to be used for the function call, since it can't multiply *n* until it knows what *fact(n-1)* is.

### Cover functions and why they're useful

Say we have an improved version of the factorial function from above that **is** tail recursive:
```Scala
@tailrec
def factTR(n: Long, p: Long): Long = if n==0 then p else factTR(n-1, n*p)
```
This one is nice, but requires us to specify what the value of p is initially, even though it's not needed for calculating the factorial. How do we fix this, especially since a user may change p to, say, 100? Easy! **Cover functions**.

A cover function allows us to manipulate what the user can provide as an input for functions. We'd do this by putting `factTR` inside a cover function. This works as it makes `factTR` local, making access and view outside the scope difficult – something we want. This also makes it easier to use, reducing our actual function call to `factc(3)`, much easier to view and use. Below is the full functional code and the result.

```Scala
 def factc(n: Long): Long =
    @tailrec
    def factTR(n: Long, p: Long): Long = if n==0 then p else factTR(n-1, n*p)
    factTR(n, 1)
    
    factc(3) // = factTR(3, 1) = factTR(2,3) = factTR(1,6) = factTR(0,6) = 6
```


## Peano's Axioms

We're bringing up Peano because we're going to make a **recursive data structure** using his argument as the basis for it. However, you may be interested in his history, as seen below.

**Giusepe Peano** (1858-1932) provided an axiomatization (the process of reducing a process to its axioms – its fundamental truths) of natural numbers, which are as follows: 

- 0 is a natural number
- For every natural number, *n*, *S(n)* is a natural number
- For all natural numbers, *m* and *n*, if *S(m)* = *S(n)* then *m* = *n*
- For every natural number *n*, *S(n)* = 0 is false
- If, for a predicate *P*, the following hold:
	- *P(0)* is true, and
	- If *P(n)* is true then it follows that *P(S(n))* is true
	then *P(n)* is true for every natural number n.

If we were to express this in something that resembles code, it would look like below:

```Python
0								    0	(zero)
S(0)						S	->	0	(one)
S(S(0))				S	->	S	->	0	(two)
S(S(S(0)))	S	->	S	->	S	->	0	(three)
etc.
```
This looks rather similar to a singly-linked list with no data stored at the nodes.

### Recursive data structures

So, to build this in Scala will require a number of things, and as such we'll go through the code section by section:

```Scala
sealed abstract class Num
case object Zero extends Num
case class Succ(n: Num) extends Num

val zero	= Zero
val one	    = Succ(Zero)
val two	    = Succ(Succ(Zero))
val three	= Succ(Succ(Succ(Zero)))
val four	= Succ(Succ(Succ(Succ(Zero))))
```
Peano suggests that there are two types of natural numbers – Zero, and non-Zero numbers. We account for this via inheritance. `object Zero` extends the `Num` abstract class, and `Succ` extends `Num`. Don't laugh, Succ = successive, so don't be immature. We use `case` before `Zero` & `Succ` because [case classes and case objects](https://www.alessandrolacava.com/blog/scala-case-classes-in-depth/) allow for pattern matching and deconstruction. You may not *need* to do this, but having the ability to do so is always appreciated. This is a **recursive aggregate pattern**. We seal the class because this allows us to restrict *where* we can define the subclasses – in this case, in the same file. 

So, now let's implement this.

```Scala
def incr(n: Num): Num = Succ(n)

// A few implementations of this are shown below:
// incr(zero)	= incr(Zero) 		= Succ(Zero)
// incr(one)	= incr(Succ(Zero))	= Succ(Succ(Zero))
```
Remember, this is essentially the successor of zero, which is 1, and the latter is 2, and so on. We can implement this into a decr definition too, but we **should consider decrementing below zero**:

```Scala
def decr(n: Num): Num = n match
  case Zero => throw new IllegalArgumentException()
  case Succ(m) => m
  
// decr(one)	= decr(Succ(Zero))		= Zero
// decr(two)	= decr(Succ(Succ(Zero))	= Succ(Zero)
```

What if we want to compare two numbers recursively? That seems complicated, but it isn't too complicated:

```Scala
def gt(m: Num)(n: Num): Boolean = (m, n) match
    case (Zero, _) => false
    case (Succ(_), Zero) => true
    case (Succ(p), Succ(q)) => gt(p)(q)
```

This can be expanded to multiplication, addition and so on. The benefit of doing this recursively is obvious – it reduces definition calls to a simple one-liner, makes reading the code much easier and simpler to write. Debugging is a different story, but you'll learn how to do that as you work with your code.

### <a name="Syntactic"></a> Syntactic sugar

Now, you might – quite fairly – suggest that stuff like `def gt` is somewhat clumsy to call. You have to call the name, then input the values that are needed. What if you wanted to make them easier to call, say turning `def gt` into `def >`? Scala lets you do that. This is referred to as **infix notation**. Below is an example using the code we've already examined prior:

```Scala
sealed abstract class Num:
    def +(that: Num): Num = add(this)(that)
    def *(that: Num): Num = mul(this)(that)
    def -(that: Num): Num = sub(this)(that)
    def **(that: Num): Num = pwr(this)(that)
    def >(that: Num): Boolean = gt(this)(that)
    
case object Zero extends Num
case class Succ(n: Num) extends Num
```

You'll notice that almost all of these new definitions *defer* to the definitions we've already made. All we've done here is make it easier to call. Whether you deem this necessary is your call, but in some cases this might be useful for you.