+++
title = "Why Did Rust Pick the 'If Let' Syntax?"
description = "Rust has a slightly idiosyncratic syntax for matching on single patterns, but there's a good reason for it!"
date = 2021-01-12

[taxonomies]
tags = ["rust", "language design"]
+++

One of my favourite features of Rust is that it has excellent support for pattern matching. This is usually done through `match` expressions, but there is also a shorter syntax that can be used if you only need a single branch, which is called `if let`.

If you're unfamiliar, a `match` expression looks like this:

```rust
let x = Some(123);

match x {
    Some(y) => {           // This matches if `x` is `Some`.
        println!("{}", y); // This prints '123'.
    }

    _ => {}                // This matches all other values.
}
```

And the equivalent `if let` expression looks like this:

```rust
let x = Some(123);

if let Some(y) = x {   // This matches if `x` is `Some`.
    println!("{}", y); // This prints '123'.
}
```

This shortcut syntax, useful as it is, can be quite confusing to new Rust developers. I commonly see people asking things like:

* Why is that `if` statement 'backwards'?
* Why is there a `let` in that conditional?
* Why do I have to use `=` instead of `==`?

I had similar questions when I started learning Rust, and to answer them, we need to take a step back and look at the plain old `let` syntax. It's relevant, I promise!

In most cases, you see `let` being used to declare a simple named variable, like this:

```rust
let x = 123;
```

What might not be apparent at first glance is that the left side of a `let` isn't *just* a name - it's a pattern, the same as what you'd use in the arm of a `match` expression!

This means that we can actually do pattern matching when declaring variables, like so:

```rust
enum Example {
    Data(i32),
}

let x = Example::Data(123); // Wrap the data.
let Example::Data(y) = x;   // Unwrap ('destructure') the data via a pattern.

println!("{}", y);          // This prints '123'.
```

There is, however, one important restriction here compared to when you're using `match` - in a `let` statement, you can only use patterns that are 'irrefutable'. In simpler terms, this means that you can only use a pattern which will match for every possible value of the type that you're matching against.

We can see what happens when this *isn't* the case by adding an extra variant to our `Example` enum:

```rust
enum Example {
    Data(i32),
    Oops,
}
    
let x = Example::Data(123);
let Example::Data(y) = x;

println!("{}", y);
```

This gives us an error which looks like this:

```rust
error[E0005]: refutable pattern in local binding: `Oops` not covered
 --> src/main.rs:8:9
  |
2 | /     enum Example {
3 | |         Data(i32),
4 | |         Oops,
  | |         ---- not covered
5 | |     }
  | |_____- `Example` defined here
...
8 |       let Example::Data(y) = x;
  |           ^^^^^^^^^^^^^^^^ pattern `Oops` not covered
  |
  = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
```

So if we can't use `let` for this pattern, what can we use? This is where we circle back to `if let`!

```rust
enum Example {
    Data(i32),
    Oops,
}
    
let x = Example::Data(123);
if let Example::Data(y) = x {
    println!("{}", y);
}
```

Notice that this example is almost identical to the previous one that didn't compile - the only changes are the extra `if`, and the extra block to mark which code should only be run if the pattern matched.

This is the reason that `if let` looks the way it does - it's an extension of the existing `let` syntax (which only supports irrefutable patterns) to support all kinds of patterns!

Once I started thinking about it this way, the syntax started to feel a lot more natural, and I hope this post helps make the relationship between `let` and `if let` click for other people too.