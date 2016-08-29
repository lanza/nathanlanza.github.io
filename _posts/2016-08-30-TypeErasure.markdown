---
layout: post
title:  "Type-erasure, protocols with associated types and generics from a simple example."
---

The other day I was writing some code and went a bit too generic happy with all my classes. A simplified version started with some code like this:

{% highlight swift %}
class NumberHolder {
   var number: Int
   weak var delegate: NumberHolderDelegate?
   func printNumber() {
      print(number)
      delegate?.didPrint(number: number)
   }
}
protocol NumberHolderDelegate: class {
   func didPrint(number: Int)
}
{% endhighlight %}

This is a pretty simple pattern. You have a type which must delegate responsibilty to some other type. This simple and contrived example gets the point across. A more practical example would be a UITableViewCell with a button on it. When the button is clicked, the cell is in no position to make any decision or to do any work for the user, so it delegates the action backwards towards it's delegate which is some UIViewController subclass.

From here, I wanted to be more "Swifty" and make my code generic for maximum reusability. I wanted NumberHolder to work for Doubles, too(or any other type, in fact.)

(Note: this code below doesn't work.)

{% highlight swift %}
class NumberHolder<Number> {
   var number: Number
   weak var delegate: NumberHolderDelegate?
   func printNumber() {
      print(number)
      delegate?.didPrint(number: number)
   }
}
protocol NumberHolderDelegate: class {
   func didPrint(number: Int)
}
{% endhighlight %}

The delegate clearly doesn't know what type of object to expect. It has to be told that it will receive a generic type.

{% highlight swift %}
protocol NumberHolderDelegate: class {
   func didPrint(number: Number)
}
{% endhighlight %}

However, it doesn't know what a "Number" is. This is where the famous associatedtype comes in.

{% highlight swift %}
protocol NumberHolderDelegate: class {
   associatedtype Number
   func didPrint(number: Number)
}
{% endhighlight %}

Great! The delegate makes sense. But now we have the famous error that confuses so many about protocols with associated types.

{% highlight swift %}
protocol 'NumberHolderDelegate' can only be used as a generic constraint because it has Self or associated type requirements
{% endhighlight %}

What does this mean? Well, the problem is that the NumberHolder type referes to a variable of the type NumberHolderDelegate. When the compiler goes to check that property, it requires to know what the associatedtype "Number" is. However, simply declaring "NumberHolderDelegate" doesn't tell the compiler this. "associatedtype Number" is a statement of "to be determined later" or "to be determined by some other means." We have to tell the compiler what that type is going to be. How? One way is through generics. The NumberHolder class turns into this:

(Note: this still doesn't work quite yet.)
{% highlight swift %}
class NumberHolder<Number, Delegate> where Delegate: NumberHolderDelegate {
   var number: Number!
   weak var delegate: Delegate?
   func printNumber() {
      print(number)
      delegate?.didPrint(number: number)
   }
}
{% endhighlight %}

We make NumberHolder generic over the type Delegate and tell the compiler that Delegate is a NumberHolderDelegate. Almost working. However, that was only half the fix. The compiler still doens't know what the associatedtype "Number" is. Here's how we tell it this information:

{% highlight swift %}
class NumberHolder<Number, Delegate> where Delegate: NumberHolderDelegate, Delegate.Number == Number {
   var number: Number!
   weak var delegate: Delegate?
   func printNumber() {
      print(number)
      delegate?.didPrint(number: number)
   }
}
{% endhighlight %}

The added clause "Delegate.Number == Number" tells the compiler that the type Delegate is (first of all a NumberHolderDelegate, but it already knew that from the first clause) declared with an associatedtype of the same type that the NumberHolder is. So upon creation of a NumberHolder:

(Note: this STILL isn't correct as the second generic type isn't being declared.)
{% highlight %}
let numberHolder = NumberHolder<Double, (Placeholder)>()
numberHolder.number = 3.14
{% endhighlight %}

the compiler can (almost) infer what is going on. First, we tell it that the instance is NumberHolder<Double,(Placeholder)>. From the first where clause, the compiler knows that Delegate is a NumberHolderDelegate. From the second clause, the compiler knows that the Delegate's associatedtype Number is going to be == to the NumberHolders generic type Number. In this case, that means the associatedtype is a Double.

However, the compiler hasn't been told exactly what object is going to take the place of the second generic type. In typical practices, you simply do someting like this:

{% highlight Swift %}
class SomeOtherClass: NumberHolderDelegate {
   func didPrint(number: Int) {
      //do something
   }
}
{% endhighlight %}

This creates a type that can serve as the delegate to the NumberHolder specific to the type Int. We can also make this generic as well:

{% highlight Swift %}
class SomeOtherClass<Number>: NumberHolderDelegate {
   func didPrint(number: Number) {
      //do something
   }
}
{% endhighlight %}

Now this class will work with any NumberHolder regardless of type. Now about this time, I realized I accidentally stumbled upon type-erasure. We just created a class (SomeOtherClass) that lets us hide away the NumberHolderDelegate from the NumberHolder and just accept any SomeOtherClass. In fact, this is where the naming "AnySuchAndSuch" comes from. We'll convert to this convention:

{% highlight Swift %}
class AnyNumberHolderDelegate<Number>: NumberHolderDelegate {
   func didPrint(number: Number) {
      //do something
   }
}
{% endhighlight %}

This enables us to remove the second generic type from NumberHolder and all it's where clauses:

{% highlight swift %}
class NumberHolder<Number> {
   var number: Number!
   weak var delegate: AnyNumberHolderDelegate<Number>?
   func printNumber() {
      print(number)
      delegate?.didPrint(number: number)
   }
}
{% endhighlight %}

This is a MUCH nicer class declaration. We traded an intermediary class, the AnyNumberHolderDelegate, for one generic type and two where clauses. 

One last caveat: this isn't EXACTLY type erasure. Typically type erasure merely copies the delegate function (or receives a closure in it's initializer) that serves as the delegate function. A truly type erased delegate would be as such:

{% highlight swift %}
class AnyNumberHolderDelegate<Number>: NumberHolderDelegate {
   var closure: ((Number) -> ())
   init(closure: ((Number) ->()) ) {
      self.closure = closure
   }
   func didPrint(number: Number) {
      closure(number)
   }
}
{% endhighlight %}

And this is our type-erased class to serve as the delegate to our NumberHolder class. Alltogether we have:

{% highlight swift %}
class NumberHolder<Number> {
   var number: Number!
   weak var delegate: AnyNumberHolderDelegate<Number>?
   func printNumber() {
      print("Number holder prints: ", number)
      delegate?.didPrint(number: number)
   }
}

protocol NumberHolderDelegate: class {
   associatedtype Number
   func didPrint(number: Number)
}

class AnyNumberHolderDelegate<Number>: NumberHolderDelegate {
   var closure: ((Number) -> ())
   init(closure: ((Number) -> ())) {
      self.closure = closure
   }
   func didPrint(number: Number) {
      closure(number)
   }
}

var numberHolder = NumberHolder<Double>()
numberHolder.number = 3.14
let numberHolderDelegate = AnyNumberHolderDelegate<Double> { number in
   print("AnyNumberHolderDelegate prints: ", number)
}
numberHolder.delegate = numberHolderDelegate
numberHolder.printNumber()

// Number holder prints: 3.14
// AnyNumberHolderDelegate prints: 3.14
{% endhighlight %}
