# JavaScript Advanced Functions: Context and Implicit Setting

## Learning Goals

* Define execution context
* Define `this`
* Access implicitly-set context in an Object-contained function expression
* Access implicitly-set context global Object in a function expression
* Prevent implicitly setting context in function calls with `use strict`
* Use available JavaScript runtimes to validate understanding

## Introduction

In the previous lesson we provided definitions of:

* "Execution Context"
* `this`
* `call`
* `apply`
* `bind`

but then built an application that didn't use them. However, with a
record-oriented application built, we will have a shared context to understand
these challenging concepts in JavaScript.

## Define Execution Context

When a function in JavaScript ***is called***, it is provided an _execution
context_.

The _execution context_ is a JavaScript `Object` that is either implicitly or
explicitly passed at the time of the function's call.

The implicit way of passing a context with a function is something we have to
memorize and accept as part of the nature of JavaScript.

The tools for explicitly passing a context at function call-time are the
methods `call`, `apply`, and `bind.`

## Define `this`

The JavaScript keyword `this` returns the current _execution context_ while the
function is being run.  Whether that context was passed explicitly or
implicitly, `this` returns it.

Some people think that `this` is a strange thing to call such an important
concept. But pronouns like "this," "he," or "here" all refer to a _context_. At
a concert, if I say (scream) "It's noisy here," you don't think "Here in the
Milky Way galaxy? I disagree.  Space has little oxygen as a sound medium and is
therefore quite quiet." Instead, you recognize the most-relevant context is at
this significant and unusual event with giant speakers and guitar players and
therefore infer that "here" refers to "this concert." JavaScript thought the
best pronoun to use was `this`, and it seems sensible to us.

## Access Implicitly-Set Context in an Object-Contained Function Expression

When a function is called, it gets an execution context passed in. That context
will be whatever the function was 'called on' - the object to the left of the
`.`  where it's called. In the below example, `byronPoodle` is to the
left of the `.`. In `byronPoodle.warn()`, `warn` gets `byronPoodle` as its
context.

```js
let byronPoodle = {
  name: "Byron",
  sonicAttack: "ear-rupturing atomic bark",
  mostHatedThing: "noises in the apartment hallway",
  warn: function() {
    console.log(`${this.name} issues an ${this.sonicAttack} when he hears ${this.mostHatedThing}`)
  }
}
byronPoodle.warn()
// LOG: Byron issues an ear-rupturing atomic bark when he hears noises in the apartment hallway
```

As you can see, `this` was set to `byronPoodle`. So, `this.name` was `byronPoodle.name` (`"Byron"`), `this.sonicAttack` was `byronPoodle.sonicAttack` (`"ear-rupturing atomic bark"`) and `this.mostHatedThing` was `byronPoodle.mostHatedThing` (`"noises in the apartment hallway"`)

A simple way of saying it: when you call `someObject.someFunction()`, the
context inside of `someFunction` will be the thing to the left of the `.`:
`someObject`.

Here's another interesting example:

```js
let speak = function() { return `It ain't easy being ${this.name}`}
let frog = { name: "Kermit" }
let pig = { name: "Miss Piggy" }
frog.speak = speak
pig.speak = speak
frog.speak === pig.speak //=> true
frog.speak()  //=> "It ain't easy being Kermit"
pig.speak()  //=> "It ain't easy being Miss Piggy"
```

Again, the crucial realization is that the context used _inside_ the function
`speak` is defined by what's "left of the dot."

This is the general behavior for understanding context-setting in JavaScript.
We'll now cover some special cases of this general behavior.

## Access Implicitly-Set Global Object in a Function Expression

What happens if we invoke a function and it's **not** defined inside an
`Object`:

```js
let contextReturner = function() {
  return this
}

contextReturner() //=> window
contextReturner() === window //=> true
```

When no object is to the left of the function, JavaScript invisibly adds **the
global object**. Thus `contextReturner` is, from JavaScript's point of view,
the same as `window.contextReturner`. You can check for yourself in the console: `window.contextReturner === contextReturner //=> true`.


A simple way of saying it: when you call `someFunction()`, the context inside
of `someFunction` will be the thing to the left of the `.`.  Since there's
nothing there, JavaScript swaps in the global object.

In browser-based JavaScript environment (or "JavaScript runtime"), the global
object is called `window`. In NodeJS, it's called `global`.

Thus, in Chrome:

```js
let locationReturner = function() {
  return this.location.host
}

locationReturner() //=> URL host serving this page e.g. developer.mozilla.org
// Implicitly: window.locationReturner(); this will be `window` in the function
```

It's worth noting that even in a function inside of another function, the inner
function's default context is still the global object:

```js

function a() {
  return function b() {
    return this;
  }
}

a()() === window //=> true
```

## Prevent Implicitly Setting a Global Object In Function Calls With `use strict`

We wish we could say that the default context was **always** the global object.
It'd make things simple.

However, in JavaScript, if the engine sees the `String` "use strict" inside a
function, it will _stop_ passing the implicit _execution context_ of the global
object.  If JavaScript sees `"use strict"` at the top of a JavaScript code
file, it will apply this rule (and other strict behaviors) to _all functions_.

```js
function looseyGoosey() {
  return this
}

function noInferringAllowed() {
  "use strict"
  return this
}

looseyGoosey() === window; //=> true
noInferringAllowed() === undefined //=> true
```

There are really no guidelines as to which you'll see more. Some programmers
think `strict` prevents confusing bugs (seems wise!); others think it's an
obvious rule of the language and squelching it is against the language's love
of functions (a decent argument!). Generally, we advise you to think of the
"default mode" as the one that permits an _implicit_ presumption of context.
For more on strict-mode, see the Resources.

### Access Implicitly-Set New Object in Object-Oriented Programming Constructor

This lesson covers how `this` is implicitly set. An important place where
`this` is implicitly set is when new instances of classes are created. Class
definition and instance creation are hallmarks of object-oriented ("OO")
programming, a style you might not be familiar with in JavaScript. Rather than
ignore this important case until later, we're going to cover it now, even
though you might not be familiar with OO programming.  If you're not familiar
with OO in JavaScript (or anywhere for that matter!), that's OK, just remember
this rule for later.  When you see `this` inside of a class definition in
JavaScript, come back and make sure you understand this rule.

It's for convenience and feels "natural" from a linguistic point of view: "The
thing we're setting up should be the default context for work during its
construction in its own function that's called `constructor`."

```js
class Poodle{
  constructor(name, pronoun){
    this.name = name;
    this.pronoun = pronoun
    this.sonicAttack = "ear-rupturing atomic bark"
    this.mostHatedThing = "noises in the apartment hallway"
  }

  warn() {
    console.log(`${this.name} issues an ${this.sonicAttack} when ${this.pronoun} hears ${this.mostHatedThing}`)
  }
}
let b = new Poodle("Byron", "he")
b.warn() //=> Byron issues an ear-rupturing atomic bark when he hears noises in the apartment hallway
```

## Use Available JavaScript Runtimes to Validate Understanding

There are two main JavaScript environments you'll encounter as you get started
with JavaScript.  Those environments are also called runtimes. They are:

1. in the browser
2. in the "shell" when running the NodeJS program

For the remainder of this module, it's very important that you not only
***read*** these lessons but that you practice, adjust, test, and explore the
material within one (or both!) of those JavaScript runtimes.

Take the initiative to own your own learning and try out these samples (with
some variation!) in a runtime of your choice.

## Conclusion

To sum up the discussion thus far:

1. Execution context is set at function call-time, implicitly or explicitly.
2. In "bare" function calls, the context is automatically set to the global object unless prevented by `"use strict"`.
3. In "non-bare" function calls, the context is automatically set to the "object to the left of the dot."
4. (For Object-Oriented JavaScript) Execution context defaults to the new thing being created in a `class`'s `constructor`

This covers the _implicit_ context-setting rules. We'll now learn about the
_explicit_ context-setting rules.

## Resources

* [strict][]

[strict]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode
