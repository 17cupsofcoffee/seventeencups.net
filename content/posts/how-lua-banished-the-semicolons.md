+++
title = "How Lua Banished the Semicolons"
date = 2018-04-03
tags = ["language design", "ein", "lua"]
+++

My current pet project outside of work is developing a little programming language called [Ein](https://github.com/17cupsofcoffee/ein). I decided fairly early on in development that I didn't want Ein to have semicolons, both as a matter of taste and as a bit of a development challenge to myself, so I've spent a fair chunk of the past week investigating how other languages make this work.

Lua's solution to this problem is (in my opinion) fairly nifty, so I thought I'd write about it on the off-chance that someone else will find it as interesting as I do 😄

## The Problem

First, some background - why is getting rid of semicolons tricky? Can't we just remove them from our language's grammar and be done with it?

The answer to this question can be summed up in one snippet of pseudo-code:

```rust
let x = 1 // Does the statement end here?
- 1       // Or does it end here?
```

How does our language's parser decide whether this should be `let x = 1; -1;` or `let x = 1 - 1;`? In the parser's eyes, they're both perfectly valid statements!

## The (Potential) Solutions

There's several ways that languages try to get around this problem. Most understandably throw their hands up and say, "Nope, not dealing with that, let's keep the semicolons". Some make the whitespace in their language significant, like Python. Others, like Go, helpfully insert the semicolons for you behind the scenes based on [a set of rules](https://golang.org/ref/spec#Semicolons).

Personally though, I'm not a fan of those solutions. Explicit semicolons are ugly to my eye, making whitespace have meaning rubs me the wrong way for reasons I don't quite understand, and automatic semicolon insertion feels like placing too much trust in the compiler to 'guess' where I meant for the statements to end.

Surely there's a way we can make things explicit without peppering our code with extra syntax?

## How Lua Does It

Turns out we can! Lua's syntax is unambigous, even if you leave out all the semicolons and nice formatting, and the main way it achieves this is by dropping a feature a lot of us take for granted - *expressions-as-statements*.

In most languages, it's perfectly valid to use an expression (a piece of code that can be evaluated to get a value, like adding two numbers together) in the same place that you would use a statement (a piece of code run for its side effects, like a variable declaration).

Lua takes a much more hardline stance on this - programs are a list of statements, some statements may contain expressions (like the condition of an `if`), but expressions are *not* allowed to be used as statements.

Let's go back to our original example, translated to Lua:

```lua
local x = 1 -- Does the statement end here?
- 1         -- Or does it end here?
```

In Lua, a variable declaration is a statement, but `-1` is an expression - therefore, the only valid way of interpreting this code is `local x = 1 - 1`!

## What's The Catch?

Ah yes, there's always a catch, and in this case it's a fairly obvious one: what if I *want* to run an expression for its side effects?

For example, a lot of the time you'll want to use the return value of a function, but sometimes you'll just want to run it. Lua caters for this scenario by  making an exception to the rule, allowing function calls to be used both in statement and expression position.

This is one of the only places that Lua bends the rules, however. In some languages, you can use the short circuting behavior of the logical AND/OR operators as short and sweet control flow statements:

```js
// JavaScript
isActive && doSomething();
```

The equivalent in Lua isn't valid unless you [assign the result to a temporary variable:](http://lua-users.org/wiki/ExpressionsAsStatements)

```lua
local _ = isActive and doSomething() -- _ has no special meaning - just a common Lua 
                                     -- naming convention for throwing away variables!
```

That said, once I started thinking about it, I realized I don't write code like that all too often! I've gone through my phase of writing [ternary](https://en.wikipedia.org/wiki/%3F:) soup, and I think I tend to prefer using more explicit/blocky syntax these days - it tends to convey my intent better. I'm starting to wonder if dropping expressions-as-statements might not be too bad a price to pay for having a completely unambiguous, semicolon-less grammar!

## Conclusion

Thank you for reading! I hope I didn't bore you to death rambling on about semicolons for $x words! ❤️

If you're interested in this kind of thing, I'd recommend taking a look at the [full grammar for Lua](http://www.lua.org/manual/5.3/manual.html#9) from its reference manual - it's really short and really clean, and there's a lot of really interesting design decisions in there which I'm interested in digging deeper into.

Now, my questions to you, dear reader:

* Do any other popular-ish languauges disallow expressions-as-statements?
* How do you feel about this solution compared to the others I mentioned? Do you think the trade-offs are worth it?
