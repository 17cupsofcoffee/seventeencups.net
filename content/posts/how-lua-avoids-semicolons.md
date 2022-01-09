+++
title = "How Lua Avoids Semicolons"
description = "Why is it that some languages require semicolons and some don't? This article attempts to explain, as well as demonstrating an interesting trick in the Lua grammar."
date = 2018-04-03
aliases = ["/posts/how-lua-banished-the-semicolons"]

[taxonomies]
tags = ["language design", "ein", "lua"]
+++

My current pet project outside of work is developing a little programming language called [Ein](https://github.com/17cupsofcoffee/ein). I decided fairly early on in development that I didn't want Ein to have semicolons, so I've spent a fair chunk of the past week investigating how other languages make this work.

Lua's solution to this problem is (in my opinion) fairly nifty, so I thought I'd write about it on the off-chance that someone else will find it as interesting as I do 😄

## The Problem

First, some background - why is getting rid of semicolons tricky? Can't we just remove them from our language's grammar and be done with it?

The answer to this question can be summed up in one snippet of pseudo-code:

```rust
let x = 1 // Does the statement end here?
- 1       // Or does it end here?
```

How does our language's parser decide whether this should be one statement (`let x = 1 - 1`) or two (`let x = 1` followed by `-1`)? In the parser's eyes, they're both perfectly valid!

## The (Potential) Solutions

There's several ways that languages try to get around this problem.  Some make the whitespace in their language significant, like Python. Others, like Go, try to insert the semicolons for you behind the scenes based on [a set of rules](https://golang.org/ref/spec#Semicolons).

Personally though, I'm not a fan of those solutions. Making whitespace have meaning rubs me the wrong way for reasons I don't quite understand, and automatic semicolon insertion feels like placing too much trust in the compiler to 'guess' where I meant for the statements to end.

## How Lua Does It

Lua's syntax is unambigous, even if you leave out all the semicolons and nice formatting, and the main way it achieves this is by dropping a feature a lot of us take for granted - *expressions-as-statements*.

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
local _ = isActive and doSomething()
-- _ has no special meaning - just a common Lua 
-- naming convention for throwing away variables!
```

## Conclusion

Thank you for reading! I hope I didn't bore you to death rambling on about semicolons for $x words! ❤️

If you're interested in this kind of thing, I'd recommend taking a look at the [full grammar for Lua](http://www.lua.org/manual/5.3/manual.html#9) from its reference manual - it's really short and really clean, and there's a lot of really interesting design decisions in there which I'm interested in digging deeper into.
