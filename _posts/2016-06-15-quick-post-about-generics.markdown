---
layout: post
title: "Quick Post About Generics"
date:   2016-06-15 1:42:0 -0400
categories: generics 
---
Just figured I'd write something simple about generics just to get this blog going.

First, I'll demonstrate the situation without generics. A Swift without generics would use structs similar to this:

{% highlight swift %}
   struct Stack {
      private var elements: [Any] = []
      func push(element: Any) {
         elements.append(element)
      }
      func pop() -> Any? {
         return elements.popLast()
      }
      init() {}
   }
{% endhighlight %}

Now, clearly, this Stack implementation is very inconvenient. If you create a stack and fill it up, it doesn't keep track of the type it holds. Worse yet, it doesn't tell you what it returns when you pop off the top element.

{% highlight swift %}
   var stack = Stack()
   stack.push(1)
   stack.push("hello")
   let someType = stack.pop()
   //someType is Any?
{% endhighlight %}

In order to use someType, you'd have to typecast it:

{% highlight swift %}
   guard let casted = someType as? String else { return }
   let uppercased = casted.uppercased()
   print(uppercased)
   //HELLO
{% endhighlight %}

This is not good programming. A (bad) solution would be to make your Stack pertain to particular types. IE we could write these two Stacks:


{% highlight swift %}
   struct IntStack {
      private var elements: [Int] = []
      func push(element: Int) {
         elements.append(element)
      }
      func pop() -> Int? {
         return elements.popLast()
      }
      init() {}
   }
   struct StringStack {
      private var elements: [String] = []
      func push(element: String) {
         elements.append(element)
      }
      func pop() -> String? {
         return elements.popLast()
      }
      init() {}
   }
{% endhighlight %}

This is, perhaps, even worse than the original situation. Now we have to write a new Stack for each and every type we need a Stack for. 

Fortunately, we have generics. A generic Stack gives us the ability to let the compiler know that the Stack will hold a particular type in it's array, take the same type for the argument for push and return the same type for pop. 

{% highlight swift %}
   struct Stack<Element> {
      private var elements = [Element]()
      func push(element: Element) {
         elements.append(element)
      }
      func pop() -> Element? {
         return elements.popLast()
      }
      init() {}
   }
{% endhighlight %}

The <Element> in Stack<Element> declares that the struct is generic with the placeholder type Element. Upon initiliazing a new Stack, you supply it the type within the angle brackets by writing Stack<Int>, for example.

{% highlight swift %}
   var intStack = Stack<Int>()
   intStack.push(3)
   intStack.push(5)
   let popped = intStack.pop()
   // popped is Int?
{% endhighlight %}

This is how Arrays, Dictionaries, Sets etc are implemented in the Swift standard library. Generics allows a struct/class author to implement a type once instead of implementing it for each and every type he'd like to use it for.
