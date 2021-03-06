# What's the difference between undefined in Haskell and null in Java?
Source: [What's the difference between undefined in Haskell and null in Java?](http://stackoverflow.com/questions/3962939/whats-the-difference-between-undefined-in-haskell-and-null-in-java)

# Answer by Don Stewart
Ok, let's back up a little.

"undefined" in Haskell is an example of a "bottom" value (denoted ⊥). Such a value represents any undefined, stuck or partial state in the program.

Many different forms of bottom exist: non-terminating loops, exceptions, pattern match failures -- basically any state in the program that is undefined 
in some sense. The value `undefined :: a` is a canonical example of a value that puts the program in an undefined state.

`undefined` itself isn't particularly special -- its not wired in -- and you can implement Haskell's `undefined` using any bottom-yielding expression. 
E.g. this is a valid implementation of undefined:

```haskell
> undefined = undefined
```
Or exiting immediately (the old Gofer compiler used this definition):

```haskell
> undefined | False = undefined
```

The primary property of bottom is that if an expression evaluates to bottom, your entire program will evaluate to bottom: the program is in an undefined state.

Why would you want such a value? Well, in a lazy language, you can often manipulate structures or functions that store bottom values, without the program being itself bottom.

E.g. a list of infinite loops is perfectly cromulent:

```haskell
 > let xs = [ let f = f in f 
            , let g n = g (n+1) in g 0
            ]
 > :t xs
 xs :: [t]
 > length xs
 2
```

I just can't do much with the elements of the list:

```haskell
> head xs
^CInterrupted.
```

This manipulation of infinite stuff is part of why Haskell's so fun and expressive. A result of laziness is Haskell pays particularly close attention to `bottom` values.

However, clearly, the concept of bottom applies equally well to Java, or any (non-total) language. In Java, there are many expressions that yield "bottom" values:

* comparing a reference against null (though note, not null itself, which is well-defined);
* division by zero;
* out-of-bounds exceptions;
* an infinite loop, etc.

You just don't have the ability to substitute one bottom for another very easily, and the Java compiler doesn't do a lot to reason about bottom values. However, such values are there.

In summary,

* dereferencing a `null` value in Java is one specific expression that yields a bottom value in Java;
* the `undefined` value in Haskell is a generic bottom-yielding expression that can be used anywhere a bottom value is required in Haskell.
That's how they're similar.

## Postscript
As to the question of `null` itself: why it is considered bad form?

* Firstly, Java's null is essentially equivalent to __adding an implicit Maybe a to every type a in Haskell__.
* Dereferencing `null` is equivalent to pattern matching for only the `Just` case: `f (Just a) = ... a ...`

So when the value passed in is `Nothing` (in Haskell), or `null` (in Java), your program reaches an undefined state. This is bad: your program crashes.

So, by adding null to every type, you've just made it far easier to create bottom values by accident -- the types no longer help you. Your language is no longer helping you prevent that particular kind of error, and that's bad.

Of course, other bottom values are still there: exceptions (like `undefined`) , or infinite loops. Adding a new possible failure mode to every function -- dereferencing `null` -- just makes it easier to write programs that crash.

