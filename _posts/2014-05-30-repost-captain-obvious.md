---
layout: default
title: "Repost: Captain Obvious on JavaScript"
---

In JavaScript, anywhere you find yourself writing:

{% highlight javascript %}
function (x) { return foo(x); }
{% endhighlight %}

You can [usually][awb] substitute just `foo`. For example, this code:

[awb]: http://www.wirfs-brock.com/allen/posts/166 "A JavaScript Optional Argument Hazard"

{% highlight javascript %}
var floats = someArray.map(function (value) {
  return parseFloat(value);
});
{% endhighlight %}

Could be written:

{% highlight javascript %}
var floats = someArray.map(parseFloat);
{% endhighlight %}

This understanding is *vital*. Without it, you can be led astray into thinking that this code:

{% highlight javascript %}
array.forEach(function (element) {
  // do something
});
{% endhighlight %}

...Is just a funny way of writing a for loop. It's not!

`forEach` isn't a way of saying "Do this thing with every member of `array`." No, `forEach` is an array method that takes a function as an argument, so this code is a way of saying "Apply every member of `array` to this function". You can pass `forEach` a function literal (as above), a variable name that resolves to a function, even an expression like `myObject.methodName` that looks like a method but is really a function defined in an object's prototype.

Once you have internalized the fact that *any* function will do, you can refactor code to clear out the cruft. This example uses [jQuery][j] in the browser:

[j]: http://jquery.org

{% highlight javascript %}
$('input').toArray().map(function (domElement) {
  return parseFloat(domElement.value);
})
{% endhighlight %}

Let's turn that into:

{% highlight javascript %}
$('input').toArray()
  .map(function (domElement) {
    return domElement.value;
  })
  .map(function (value) {
    return parseFloat(value);
  })
{% endhighlight %}

And thus:

{% highlight javascript %}
$('input').toArray()
  .map(function (domElement) {
    return domElement.value;
  })
  .map(parseFloat)
{% endhighlight %}
Once you get started turning function literals into other expressions, you can't stop. The next step on the road to addiction is using functions that return functions:

{% highlight javascript %}
function get (attr) {
  return function (object) { return object[attr]; }
}
{% endhighlight %}

Which permits us to write:

{% highlight javascript %}
$('input').toArray()
  .map(get('value'))
  .map(parseFloat)
{% endhighlight %}

Which really means, "Get the `.value` from every `input` DOM element, and map the `parseFloat` function over the result."

Obviously!

## Part II: Less obvious but still interesting

Although the example illustrated the obvious points about functions as first class entities, the final example involved creating a new function and iterating *twice* over the array. Avoiding the extra loop may be an important performance optimization. Then again, it may be premature optimization. But either way, once we have absorbed the obvious, we're ready to look at the practical.

We might express our discomfort thus: "We wish to decompose an expression into functions. Our obvious example recomposed them into two functions and two maps, but for performance reasons we would like to compose two functions and only one map."

As usual, finding the right question to ask is half the battle. Familiarity with good libraries is the other half. For our purposes, `Functional.sequence` from Oliver Steele's [Functional JavaScript][fj] library will be useful. `Functional.sequence` composes two or more functions in the argument order (Functional Javascript also provides `.compose` to compose multiple functions in applicative order). For the purpose of this post, they could be defined something like this:

[fj]: http://osteele.com/sources/javascript/functional/

{% highlight javascript %}
var naiveSequence = function (a, b) {
  return function (c) {
    return b(a(c));
  };
}

var naiveCompose = function (a, b) {
  return function (c) {
    return a(b(c));
  };
}
{% endhighlight %}

(`Functional.sequence` and `Functional.compose` are far more thorough than these naive examples, of course.)

Given this, you could expect that:

{% highlight javascript %}
Functional.sequence(get('value'), parseFloat)({ value: '1.5' })
  // => 1.5
{% endhighlight %}

Thus, we can rewrite our map as:

{% highlight javascript %}
$('input').toArray()
  .map(
    Functional.sequence(
      get('value'),
      parseFloat
    )
  )
{% endhighlight %}

Now the code iterates over the array just once, mapping it to the composition of the two functions while still preserving the new character of the code where the elements of an expression have been factored into separate functions. Is this better than the original? It is if you want to refactor the code to do interesting things like memoize one of the functions. But that is no longer obvious.

What *is* obvious is that JavaScript is a functional language, and the more ways you have of factoring expressions into functions, the more ways you have of organizing your code to suit your own style, performance, or assignment of responsibility purposes.

p.s. Captain Obvious would not write such excellently plain-as-the-nose-on-his-face posts without the help of people like [@jcoglan](https://twitter.com/#!/jcoglan), [@CrypticSwarm](https://twitter.com/#!/CrypticSwarm), [@notmatt](https://twitter.com/#!/notmatt), [@cammerman](https://twitter.com/#!/cammerman), [Skyhighatrist](http://www.reddit.com/user/Skyhighatrist), and [@BrendanEich](https://twitter.com/#!/BrendanEich).

p.p.s. This post first appeared in January of 2012, on my old blog. I may repost other technical material, if and when time permits.

[pi]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/parseInt
[pf]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/parseFloat
[vnl]: http://crypticswarm.com/variables-now-and-later
[sicp]: http://mitpress.mit.edu/sicp/
[method-combinators]: https://github.com/raganwald/method-combinators
[Method Combinators in CoffeeScript]: https://github.com/raganwald/homoiconic/blob/master/2012/08/method-decorators-and-combinators-in-coffeescript.md#method-combinators-in-coffeescript