---
layout: default
title: "AngularJS and scope.$apply"
---

If you've written a non-trivial amount of code in AngularJS, you may
have come across the `$scope.$apply()` method. On the surface, it may
seem like just a method you call to get your bindings to update. But
why does it exist? And when do you need to use it?

To really understand **when** to use `$apply`, it's good to know exactly **why**
we need to use it, so let's dive in!

## JavaScript is Turn Based

The JavaScript code we write doesn't all run in one go, instead it
executes in turns. Each of these turns runs uninterupted from start to
finish, and when a turn is running, nothing else happens in our
browser. No other JavaScript code runs, and our web page interface is
completely frozen. This is why poorly coded JavaScript can freeze a
web page.

Instead, whenever there is a task that takes some amount of time, such
as an Ajax request, waiting for a click event, or setting a timeout, we
set up a callback function and finish our current turn. Later, when the Ajax
request completes, a click is detected, or the timer completes, a new
JavaScript turn is created and the callback is run to completion.

Let's look at an example JavaScript file:

    var button = document.getElementById('clickMe');

    function buttonClicked () {
      alert('the button was clicked');
    }

    button.addEventListener('click', buttonClicked);

    function timerComplete () {
      alert('timer complete');
    }

    setTimeout(timerComplete, 2000);

When the JavaScript code is loaded, that is a single turn. It finds a
button, adds a click listener, and sets a timeout. Then the turn is
complete, and the browser will update the web page if necessary, and
begin accepting user input.

If the browser detects a click on `#clickMe`, it creates a new turn,
which executes the `buttonClicked` function. When that function returns,
that turn is complete.

After 2000 milliseconds, the browser creates a new turn which calls
`timerComplete`. 

Our JavaScript code is run in turns, and in between the turns is when
the page is repainted, and input is accepted.

## How do we update bindings?

So Angular lets us bind parts of our interface to data in our JavaScript
code, but how does it know when data changes, and the page needs
updating?

There are a few solutions. The code needs to know when a value has
changed. Right now there is no way for our code to be directly 
notified of changes on an object [^1]. Instead there are two main
strategies.

One strategy is to use special objects, where data is set via methods,
not property assignments. Then changes can then be noted, and the page can be
updated. This has the downside in that we must extend some special
object. Also, for assigning, we must use a more verbose form
`obj.set('key', 'value')` instead of  `obj.key = 'value'`. Frameworks
like [EmberJS](http://emberjs.com) and
[KnockoutJS](http://knockoutjs.com) use this strategy.

AngularJS takes a different approach: allow any value to be used as
a binding target. Then at the end of any JavaScript code turn, check to
see if the value has changed. This may seem inneficient at first, but
there are some clever strategies to reduce the performance hit. The big
benefit is we can use normal objects and update our data however we
want, and the changes will be noticed and reflected in our bindings.

For this strategy to work, we need to know when data has possibly
changed, and this is where `$scope.$apply` comes into play.

## $apply and $digest

That step that checks to see if  any binding values have changed actually
has a method, `$scope.$digest()`. That's actually where the magic
happens, but we almost never call it directly, instead we use
`$scope.$apply()` which will call `$scope.$digest()` for you.

`$scope.$apply()` takes a function or an Angular expression string, and
executes it, then calls `$scope.$digest()` to update any bindings or
watchers. 

So, when do you need to call `$apply()`? Very rarely, actually.
AngularJS actually calls almost all of your code within an $apply call.
Events like `ng-click`, controller initialization, `$http` callbacks are
all wrapped in `$scope.$apply()`. So you don't need to call it yourself,
in fact you can't. Calling $apply inside $apply will throw an error.

You do need to use it if you are going to run code in a new turn. And
only if that turn isn't being created from a method in the AngularJS
library.
Inside that new turn, you should wrap your code in `$scope.$apply()`.
Here is an example. We are using `setTimeout`, which will execute a
function in a new turn after a delay. Since Angular doesn't know about
that new turn, the update will not be reflected.

<iframe style="width: 100%; height: 200px"
src="http://jsfiddle.net/xsPKK/1/embedded/js,html,result"
allowfullscreen="allowfullscreen" frameborder="0"> </iframe>

But, if we wrap the code for that turn in `$scope.$apply()`, the change
will be noticed, and the page is updated.

<iframe style="width: 100%; height: 200px"
src="http://jsfiddle.net/m3tsA/1/embedded/js,html,result"
allowfullscreen="allowfullscreen" frameborder="0"> </iframe>

_As a convenience, AngularJS provides
[$timeout](http://docs.angularjs.org/api/ng.$timeout), which is like
`setTimeout`, but automatically wraps your code in $apply by default. Use that,
not this_

If you write any code that uses Ajax without `$http`, or listens for
events without using Angular's `ng-*` listeners, or sets a timeout
without `$timeout`, you should wrap your code in `$scope.$apply`

## $scope.$apply() vs $scope.$apply(fn)

Sometimes I see examples where data is updated, and then
`$scope.$apply()` is called with no arguments. This achieves the desired
result, but misses some opportunities.

If your code isn't wrapped in a function passed to $apply, and it throws
an error, that error is thrown outside of AngularJS, which means any error
handling being used in your application is going to miss it. $apply not
only runs your code, but it runs it in a `try/catch` so your error is
always caught, and the $digest call is in a `finally` clause, meaning it will
run regardless of an error being thrown. That's pretty nice.


Hopefully now you understand what `$apply` is and when to use it. If you
only use what AngularJS provides you, you shouldn't need to use it often. But
if you begin writing directives where you are observing DOM elements
directly, it is going to become necessary.

[^1]: Object.observe has been proposed for ES5, but is only
experimentally implemented now



