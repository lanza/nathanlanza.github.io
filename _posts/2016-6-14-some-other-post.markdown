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
	}
{% endhighlight %}

Lastly, here's it in action.

{% highlight swift %}
	let muffin = Dog(name: "Muffin", age: 12)
	print(muffin.name)
	//Muffin
{% endhighlight %}

That'll do, pig.
