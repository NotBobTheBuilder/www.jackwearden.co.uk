# Bending JavaScript with Type Coercion

## Reducing JavaScript to ungoogleable characters

Sometimes, one line of code is all you need. The JavaScript

    alert('just banter m80');

when run, opens an alert message with the text “just banter m80”.
An odd choice of words, but the process is clear.

The only code quality metric I’ve ever believed in is [WTFs per minute](http://www.osnews.com/story/19266/WTFs_m), and were I reviewing my code, I’d probably give it a 0.

As far as I’m aware, there’s no upper bound of WTFs per minute, so I thought I’d try and find one.

The end result is [here](https://github.com/NotBobTheBuilder/stupidest_thing/blob/master/lib/dumb.js), but I encourage you to read the process before looking at it.

## Building Blocks

JavaScript’s worst part, in my opinion, is it’s implicit type coercion. It does really weird things. For example:

    // string + number
    "hello" + 1 == "hello1"

Number gets cast to a string, and appended. Perhaps undesirable, but not too weird. Then:

    // array + array
    [] + [] == ""

Two added arrays are casted to strings and joined. What. Also:

    // array + Object
    [] + {} == "[object Object]"

That’s weirder - an object is cast to the string "[object Object]". Definitely weird. In fact, all JS types can be cast into strings - normally meaninglessly. This is very useful for us later, and is 1 of 2 building blocks for what we do later. The other building block is that in modern JavaScript runtimes, strings are indexable by number. That is to say `"hello world"[0] == "h"`.
## Building something crazy.

So, we start with `alert('just banter m80')`. Lets expand that into something that only uses ungooglable characters.

The obvious place to start is the message itself. We know we can build strings by joining the results of weird casts together, so lets start with this:

    ([] + {} + {}[{}] + ![] + !![])

This hideous piece of code gives us the string `[object Object]undefinedfalsetrue` and works like this:

    ([]       // by starting with an array, everything that follows will be cast to string

    + {}      // {}.toString() == '[object Object]'

              // this accesses an object property - the inner {} is cast to "[object Object]"
    + {}[{}]  // there is no [object Object] property in an empty object, so this gives us undefined
              // undefined, when cast to undefined, is simply "undefined"

    + ![]     // ![] casts an empty array to an empty string, which is then cast to boolean, and is false

    + !![]    // we get to false by the same rule as above, and !false is true.
    )

Success! We now have the letters "abcdefijlnorstu", which is pretty much all of the letters in our message.

But how do we get at them? And how do we build our `80`? Fortunately, both are solvable with Numbers - we can access individual letters in a string by index, and numbers are trivially castable to strings.


## Numbers

Another “nuance” of JavaScript is how we can make numbers by adding booleans. We can get 1 by adding `false` and `true`:

    ![] + !![] == 1

so we can quickly grab some powers of 2 like this:

    _=![] + !![],     // 1
    __=_+_,           // 2
    ___=__*__,        // 4
    ____=___*__,      // 8
    _____=____*__     // 16

and if we save our nasty string too:

    ______=([] + {} + {}[{}] + ![] + !![])

we can start grabbing individual letters like so:

    // [obj…
    //    ^ 3

    ______[_+__] // j (3)

And now when we call alert, we can use everything we designed so far:

    alert(
        ______[_+__]                // j (03)
      + ______[_____-_]             // u (15)
      + ______[(_____*__)-___-_]    // s (27)
      + ______[___+__]              // t (06)
      + ______[____-_]              //   (07)
      + ______[__]                  // b (02)
      + ______[_____+____+_]        // a (25)
      + ______[_____]               // n (16)
      + ______[___+__]              // t (06)
      + ______[___]                 // e (04)
      + ______[_____*__-__]         // r (30)
      + ______[____-_]              //   (07)
      + 'm'
      + ____                        // 8
      + (_-_)                       // 0
    );

You’ll notice we can’t get an m from our existing string. This isn’t too tricky to solve.

## Constructors

Every “thing” in JavaScript (that is, everything that’s not `null` or `undefined`) has a *constructor*. A constructor is a function that can be used to make values of that type.

For example, `false.constructor` is the `Boolean` function, `"".constructor` is the `String` function and, most usefully, `(0).constructor` is the `Number` function. That’s right - Number with a big, lovely "m". can

And, would you believe it, we have all the letters we need to build the string "constructor":

    (_[
        ______[___+_]               // c (05)
      + ______[_]                   // o (01)
      + ______[_____]               // n (16)
      + ______[(_____*__)-___-_]    // s (27)
      + ______[___+__]              // t (06)
      + ______[_____*__-__]         // r (30)
      + ______[_____-_]             // u (15)
      + ______[___+_]               // c (05)
      + ______[___+__]              // t (06)
      + ______[_]                   // o (01)
      + ______[_____*__-__]         // r (30)
    ]) // Number function

Now we have the function, we just need the m. Functions can be cast to strings too - if we pull out the now-clichéd technique of prepending with an array, we get this:


    [] + Number == "function Number() { [native code] }"

(the `[native code]` block is because Number is implemented in C, not JS)

all we need to do now is get the m out from index 11 (____+___-_).

The full code at this point is [here](https://github.com/NotBobTheBuilder/stupidest_thing/blob/v2/lib/dumb.js); but the core part is here:

    alert(
        ______[_+__]                // j (03)
      + ______[_____-_]             // u (15)
      + ______[(_____*__)-___-_]    // s (27)
      + ______[___+__]              // t (06)
      + ______[____-_]              //   (07)
      + ______[__]                  // b (02)
      + ______[_____+____+_]        // a (25)
      + ______[_____]               // n (16)
      + ______[___+__]              // t (06)
      + ______[___]                 // e (04)
      + ______[_____*__-__]         // r (30)
      + ______[____-_]              //   (07)
      + ________                    // m
      + ____                        // 8
      + (_-_)                       // 0
    );

The next question is though - how do we get rid of `alert`? It’s not a string or value at this point, it’s a variable name.

It’s here that the rabbit hole deepens.

## Alert

Like many interpreted languages, JavaScript has an `eval` function, which takes in a string and runs it as code. Since we have the letters to build the word `alert`, `eval('alert')` would give us the `alert` function. But how do we get eval? We’d have to `eval('eval')`, which doesn’t help at all.

The answer is along the same lines though. Functions have a constructor called `Function`, which accepts a string and returns a function with that string as a body. So if we run

    Function('return alert')

we get a function which gives us the alert function, and in turn

    Function('return alert')()

is the alert function itself.

But where do we get the `Function` value from? Since all functions have `Function` as a constructor, any function will do - and fortunately we have one - `Number`!

Unobfuscated, then, our code looks like this:

    0['constructor']['constructor']('return alert')()('just banter m80')

It works like this:

    0['constructor']    // Number
    ['constructor']     // Function
    ('return alert')    // A function that returns alert
    ()                  // the alert function itself
    ('just banter m80') // alert with our message

Now if we pull together our string construction from type coercion, we get this:

    _=![] + !![],     // 1
    __=_+_,           // 2
    ___=__*__,        // 4
    ____=___*__,      // 8
    _____=____*__,    // 16
    ______=([] + {} + {}[{}] + ![] + !![]), // [object Object]undefinedfalsetrue
    _______=  // "contructor"
        ______[___+_]               // c (05)
      + ______[_]                   // o (01)
      + ______[_____]               // n (16)
      + ______[(_____*__)-___-_]    // s (27)
      + ______[___+__]              // t (06)
      + ______[_____*__-__]         // r (30)
      + ______[_____-_]             // u (15)
      + ______[___+_]               // c (05)
      + ______[___+__]              // t (06)
      + ______[_]                   // o (01)
      + ______[_____*__-__],        // r (30)
    ________= _[_______], // Number function
    _________=([] + ________)[____+___-_], // m from Number.toString()
    ________[_______](
        ______[_____*__-__]         // r (30)
      + ______[___]                 // e (04)
      + ______[___+__]              // t (06)
      + ______[_____-_]             // u (15)
      + ______[_____*__-__]         // r (30)
      + ______[_____]               // n (16)
      + ______[____-_]              //   (07)
      + ______[_____+____+_]        // a (25)
      + ______[_____+____+__]       // l (26)
      + ______[___]                 // e (04)
      + ______[_____*__-__]         // r (30)
      + ______[___+__]              // t (06)
    )()(
        ______[_+__]                // j (03)
      + ______[_____-_]             // u (15)
      + ______[(_____*__)-___-_]    // s (27)
      + ______[___+__]              // t (06)
      + ______[____-_]              //   (07)
      + ______[__]                  // b (02)
      + ______[_____+____+_]        // a (25)
      + ______[_____]               // n (16)
      + ______[___+__]              // t (06)
      + ______[___]                 // e (04)
      + ______[_____*__-__]         // r (30)
      + ______[____-_]              //   (07)
      + _________                   // m
      + ____                        // 8
      + (_-_)                       // 0
    );

Beautiful.

You may wonder why I’m using commas instead of semicolons. Because you can’t google it.

## Contributing

I’ll happily merge pull requests if the changes make the code more complex, or make me feel nauseous.
