---
layout: post
title:  "What do people talk about on blogs?"
date:   2016-06-14 01:35:00 -0400
categories: test 
---
I should write a tutorial or something. 

Here's how you create a protocol in Swift.

{% highlight swift %}
	protocol Animal {
		var name: String { get set }
		var age: Int { get set }
	}
{% endhighlight %}

And here's how you use it.

{% highlight swift %}
	struct Dog {
		var name: String
		var age: Int
      func bark() {
         print("Woof")
      }
	}
{% endhighlight %}

So, why  would we want to use protocols? Well, if you are writing some code that only requires the interface provided by the protocol, you can ignore the specifics of the struct/class that implements it. So, quickly, we'll add a second type that conforms to the protocol.

{% highlight swift %}
   struct Alligator {
      var name: String
      var age: Int
      func swim() {
         print(name, " is swimming")
      }
   }
{% endhighlight %}

Now, if we have a function that does some action particular to the name and age of an object, we can be less specific than either Dog or Alligator and just use Animal.

{% highlight swift %}
   func printAgeAndNameOf(animal: Animal) {
      print("\(animal.name) is (animal.age) years old.")
   }
   let animals: [Animal] = [Dog(name: "Muffin", age: 12), Alligator(name: "Tim", age: 4)]
   for animal in animals {
      printAgeAndNameOf(animal: animal)
   }
   // Muffin is 12 years old.
   // Tim is 4 years old.
{% endhighlight %}

That'll do, pig.
