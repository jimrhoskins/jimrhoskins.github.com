---
layout: default
title: "Nested Scopes in AngularJS"
---

If you haven't yet watched, Miško Hevery's talk on [AngularJS Best
Practices](http://www.youtube.com/watch?v=ZhfUv0spHCY) is well worth the watch. It's jammed packed
with excellent insights on how to use AngularJS effectively.

This post is a deep-dive into a specific point about nested scopes and your data
model. This point begins at
[29:19](http://www.youtube.com/watch?v=ZhfUv0spHCY&t=29m19s) and a
couple of phrases are used as guidelines (paraphrasing) "The scope is
not the model, the scope **refers** to the model" and "Whenever you 
have ng-model there's gotta be a dot in there
somewhere. If you don't have a dot, you're doing it wrong."

He uses a live example to illustrate the point. Below is that example.
To see the behavior, first manipulate the second input, and notice all
three references are updated as expected. Then edit the first field,
then the second field again. Now the first field is disconnected from
the second field and final text output.

<iframe style="width: 100%; height: 300px"
src="http://jsfiddle.net/CmXaP/embedded/result,html,js"
allowfullscreen="allowfullscreen" frameborder="0"> </iframe>


Now if you are familiar with the way prototypal inheritance works in
JavaScript, and  you also know that scopes use prototypal inheritance when
creating child scopes, it should be pretty clear what this all means.
 If not, it can be pretty confusing, so
let's take a crash course on prototypal inheritance.

## Prototypal Inheritance 

JavaScript uses a very simple way to deal with inheritance. We won't dig to deep into it,
but let's get a mental model of how it works.

An object can be thought of as a bag of key/value pairs. If I have the
following code

    var jim = {
      first_name: "Jim",
      last_name: "Hoskins",
      company: "Treehouse" 
    }

Jim is an object with the key `first_name` mapped to the value `"Jim"`,
`last_name` mapped to `"Hoskins"` and `company` mapped to `"Treehouse"`.
So I can refer to `jim.first_name` and get the value, or assign to
`jim.first_name` to set the value for that key.

But here's something interesting, I can call `jim.toString()` even
though I didn't define `toString` on my object. That's because each
object has an invisible link to one other object, its prototype, or
parent.

When I execute `jim.toString()`, it first needs to find the key/value
pair for `toString`. It looks in the `jim` object, but it clearly
doesn't exist. So it automatically follows that link to the prototype to
see if `toString` exists there, and it does. In this example, `jim`'s
prototype is `Object.prototype`, and you can actually find `toString`
directly at `Object.prototype.toString`.


But here's the thing, once If I assign to `jim.toString` like so:

    jim.toString = function () {
      return "I am Jim!";
    }

I am creating a `toString` key directly on `jim`, so calling
`jim.toString()` now no longer needs to search up the prototype to find
it. This turns out to be a really useful mechanism, because it allows
many objects to share, or inherit, certain properties or methods, which
means each object doesn't need its own copy of the value. This also
allows us to override or overwrite any value we have inherited, without
messing anything up higher up.

So reading a value will search the parents if it is not found, but
writing a value always writes directly to the object, even if it is also
defined higher up.

## Angular Scopes and Prototypes.

AngularJS will often create child scopes inside of directives, for the
same reasons we saw above: it allows a directive to have it's own space
for key/value pairs, without messing with higher level data. Child
scopes inherit from parent scopes using prototypal inheritance. (There
is an exception for isolated scopes);

But we often do want to transparently write to a higher level scope, and
this is where the "dot rule" comes in. Let's see how that is applied.


Do not do this

    // Controller 
    $scope.first_name = "Jim"

    // HTML
    <input ng-model="first_name">

That is bad because we can't write to it from a child scope, because a
child scope uses prototypal inheritance. Once you write to
`first_name` from a child scope, it would become a distinct value in
that child scope, and no longer refer up to the parent's `first_name`.

Instead, do this

    // Controller
    $scope.person = {
      first_name:  "Jim"
    }

    // HTML
    <input ng-model="person.first_name">


The difference is we are not storing the data directly on the scope. Now
when ng-model wants to write, it actually does a read, then write. It
reads the `person` property from the scope, and this will work from a
child scope because a child scope will search upwards for a read. Once
it finds `person`, it writes to `person.first_name`, which is no
problem. It just works.

Play with the updated example here to see it in action.

<iframe style="width: 100%; height: 300px"
src="http://jsfiddle.net/vKuuk/1/embedded/result,html,js"
allowfullscreen="allowfullscreen" frameborder="0"> </iframe>

Hopefully you now have a better grasp on the technical reason this type
of code behaves the way it does. Quite often, bugs can be solved by
simply nesting your property in another object. That little bit of
indirection can be very powerful.

