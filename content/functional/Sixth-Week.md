---
title: "Sixth Week"
#description: <descriptive text here>
date: 2023-02-20T17:41:14Z
draft: true
toc: false
image: ""
tags: []
series: "Functional Programming"
type: "Functional"
---

# Sixth Week – Pictures

## How do we make pictures using code?

There are a multitude of ways to make a picture using code – inserting PNGs, JPG etc. is one method and one you're familiar with if you've ever written a section of HTML code. However, in this course, we'll compose pictures using **Seq** (essentially a list) & **Vector** collection types. This allows us to:

- Make ASCII art,
- Combine pictures to make new ones or modify them via various transformation methods
- Present data easily

And a lot more. We're going to do this through the creation of a library, making it much easier to re-use later on. For the sake of outlining our requirements, our library must:

- [X] Have the ability to compose pictures using picture operators.
- [X] Use default parameters.
- [X] Apply higher-order functions to solve non-trivial issues.
- [X] Use a companion object in which [Static](https://alvinalexander.com/scala/how-to-static-members-in-scala-companion-objects-fields-methods/) members are collected.


### What will our code look like?

Visualising this in your head may prove rather difficult (it is for me), so below will be an example of how our program will *hopefully* operate: 

```Scala
val matrix = Picture("01234\n56789")
val space  = Picture(' ')

println(matrix.frameNC + space + matrix.transpose.frameNC)
```

The hope is that it will generate a visually appealing table that we can appreciate, such as the one shown below:

```Python
-----   -- 
|01234| |05|
|56789| |16|
 -----  |27|
        |38|
        |49|
         -- 
```

There will be more methods than this, but this shows a very brief example of what we're looking for.

## Structuring our picture library

In almost any language, a library needs a given structure and design in order to be used. This is no different, and as such we'll create the class and object that's needed. I'll provide the code below to view, but explain it afterwards:

```Scala
package lib.picture

class Picture(cells: Seq[Seq[Char]]) {
  import Picture.*
  val depth: Int = … // Number of lines in picture
  val width: Int = … // Length of the longest line
  val text = … // A copy of cells whereby each line is padded by width.
  def isEmpty: Boolean = dept == 0
  def notEmpty: Boolean = ! isEmpty
  // etc.
}
```
Much like in HTML, the `depth`, `width` and `text` represent the state of Picture. The definitions exist in each instance of Picture, and apply when their conditions are met. If you've ever made a class in literally any other language, this should look rather familiar.

It should be noted that pictures **must** be rectangular in this case, which means we have to account for input that doesn't conform to this requirement. This is where *validation* and *normalisation* of input becomes pivotal.

```Scala
object Picture {
  val space: Char = ‘ ‘
  // etc.
  def apply(): Picture = new Picture(Vector())
  val empty: Picture = apply()
  def flow(width: Int, s: String): Picture = {
    …
  }
  // etc.
}
```

This object is our **companion object**. [Scala's companion objects](https://www.educba.com/scala-companion-object/) are a great way of encapsulating code for re-use later. Furthermore, it allows us to access the class' private members through the use of definitions i.e. `def apply()` is able to access the Picture class. You should remember that these are **not** instance methods – essentially methods that perform a specific task on their own. These *belong* to the object Picture, similar to static methods in Java.

### How do we validate inputs?

In other languages, for-loops and switch cases would become your best friend here. But you chose to use a Functional Programming language, meaning you get to be special. Below is the transformed version of the class that considers malformed inputs.

```Scala
class Picture(cells: Seq[Seq[Char]]) {
  val depth: Int	= cells.length
  val width : Int 	= depth match
    			      case 0 => 0
    			      case _ => (cells map (_.length)).max
    			      
  val text: Vector[Vector[Char]] = width match
    case 0 => Vector()
    case _ => cells.toVector.map
			(_.padTo(width, Picture.space).toVector)
```

Say we made a Picture, such as `new Picture( List( List(‘f’, ‘o’, ‘r’, ‘t’, ‘y’), List(‘t’, ‘w’, ‘o’) ) )`. How would this look in the class? 