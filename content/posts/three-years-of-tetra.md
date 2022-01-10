+++
title = "Three Years of Tetra"
description = "A retrospective on the development of Tetra, a 2D game framework written in Rust."
date = 2022-01-10

[taxonomies]
tags = ["rust", "gamedev"]
+++

Happy new year!

It's been just over three years since I released the first version of [Tetra](https://github.com/17cupsofcoffee/tetra), my Rust 2D game framework. I also [recently announced on Twitter](https://twitter.com/17cupsofcoffee/status/1479601522661109764) that the next version, 0.7, is probably going to be the last one that I work on for the foreseeable future.

With both of these things in mind, I thought it'd be good to take a look back at what went well with Tetra, what didn't go so well, and what I'm planning to do next.

## The good

Let's start with the good stuff - and there's a lot!

### People actually used Tetra?!

This was by far the most surprising (and the most rewarding) part of the whole project. I fully expected absolutely no-one to pay attention to Tetra, but within a few months of the first release, I started seeing people actually make stuff with it!

Not only was this extremely exciting, it was massively educational - this was the first time any project of mine had actually been used by anyone, and working through their questions, bugs and feature requests forced me to learn a lot, both about graphics programming and about managing an open source project.

I have to give special mention to [puppetmaster](https://puppetmaster.itch.io/) here - he was the earliest adopter of Tetra, using it for loads of really cool game jam projects (I spent hours playing [Shoot Out Your Life](https://puppetmaster.itch.io/shoot-out-your-life)!) and providing a ton of incredibly useful feedback. I don't think I would have gotten nearly as far with it as I did without his support, and I appreciate it a lot!

### The Rust game dev community is great

The Rust game development community has consistently been one of the most friendly and inclusive online spaces I've been involved in, and this especially goes for the [Game Development in Rust](https://discord.gg/yNtPTb2) Discord server. I cannot recommend it enough to anyone who's interested in making games in Rust - there's a genuine sense of community and camaraderie there despite it having over a thousand members, and no matter how simple or complex the question, someone always comes back with a really well-thought out answer.

*just don't get them started on color spaces again, please, i beg you*

That friendly atmosphere ripples through the projects that use Rust, as well - over the years I've been working on Tetra, I genuinely can't remember a bad interaction I've had with a user or contributor. That's not something to be taken for granted, given how toxic/entitled the atmosphere around open source can sometimes be.

### The documentation is pretty good

If you'll excuse me tooting my own horn just a little bit... I'm very proud of the effort I put in to documenting Tetra, and I think it's one of the most beginner-friendly engines in the ecosystem!

I think the key factor in this is that I actually really enjoy writing documentation, and it was a bit eye-opening to talk to other devs in the community and realize that isn't the case for a lot of (or even most?) people. It's very easy to chalk bad documentation up to laziness, but especially in open-source projects where people are just contributing their spare time, I don't think it's reasonable or realistic to expect every single developer to absolutely love writing. Instead, I think projects need to do a better job of emphasising the importance of non-code contributions, and seeking out people who actually have an interest in that kind of work.

That's easier said than done though - one of my many new years resolutions is to raise PRs when I run into bad docs instead of griping on Twitter :p

## The bad

So, given how positive I've just said working on Tetra was, why do I now want to stop? I think there's a few reasons; two technical and one more personal.

### The design is flawed

Tetra was my first serious Rust project, and my first *ever* graphics programming project. In fact, I'm pretty sure the folder was still called `opengl_tutorial` when I published the first commit...

While I think I've done okay considering, there's a lot of design decisions I made when I was less experienced that are now coming back to bite me:

* There's too much global state in the renderer, and it makes it really hard to write composable code on top of it. For example, [`egui-tetra`](https://github.com/tesselode/egui-tetra) leaves your blend mode and scissor rectangle in a completely different state after calling into it, and there's no way for the developer of that library to do anything about it.
* The abstractions for text rendering, animation and screen scaling are really limited and hard to extend without making changes to the engine's internals.
* The game loop is designed in a way that makes zero sense on the web, as error reporting is done synchronously through function returns.
* There's a lot of places where there has to be a seperate internal and external API for the same thing due to borrow checking issues with `Context`, and it feels gross.
* The list goes on...

I have ideas about how I'd fix these issues, but to do so, I'd need to basically rewrite half of the framework, and the thought of that just fills me with dread. Do I really have the energy to spend another *X* years on this, just to get back to feature-parity with where I am now? And would the end result even be recognizable as Tetra? A clean break feels like the least stressful option to me.

### General-purpose game engines are hard to extend

Towards the end of last year, I ran into an issue while working on a game prototype. Tetra's TTF renderer generated non-premultiplied textures, while my project drew everything with premultiplied alpha, so all my text was being drawn as white squares. I was able to work around this by changing the blend mode midway through drawing, but that felt hacky - surely I could fix things in Tetra itself?

If I were working on a private engine, I could just tweak the texture generation code, and the job would be done. But because Tetra is public and other people are relying on it, I had to come up with a whole new API for switching between the old behaviour and the new behaviour, I had to think about consistency with the other font formats, I had to think about whether I should change the defaults, and so on and so forth. What should have been a one line change turned into multiple days of bikeshedding, with my actual use case falling by the wayside.

This was one example of something that happened *constantly* while I was working on Tetra. Seemingly innoccuous features would explode in complexity due to the fact I was trying to cater to every possible use case while maintaining backward compatibility, and it just left me feeling exhausted.

I'll admit that some of this was self-inflicted - I have a bit of a perfectionist streak, often to my own detriment - but I also think this is just something that comes with the territory when you try to make an engine that's all things to all people. You end up making compromises everywhere, and the end result is much less coherent than if you'd just tried to do one thing well.

### Engine development is procrastination

Here's the *big oof* for me, looking back on the last three years: my original goal when I started working on Tetra was to make tools that would make me more productive as a game developer, and if we're measuring by that metric, it's been an abject failure. I've released a grand total of [one game](https://17cupsofcoffee.itch.io/lonely-star) since I started working on this project, and that was nearly two years ago now. So what went wrong?

I think it's this: writing engine code was a really good way to trick myself into thinking I was being productive without actually making any games. *Sure, I've not touched my RPG project in months, but look, canvases support multisampling now,* I'd tell myself, completely ignoring the fact that I exclusively make 2D pixel art games that are never going to use it. This was exacerbated by the fact that Tetra was being used by other people - someone would come to me with an interesting feature request and I'd drop everything to implement it.

I think this is a really easy trap for would-be game developers to fall into, especially if you've got the kind of brain that easily gets sidetracked by fun technical puzzles. I know it's a played out saying at this point, but I can't stress it enough - if you want to write games, then [write games, not engines](https://geometrian.com/programming/tutorials/write-games-not-engines/). Your code will be simpler, and you'll have a lot more to show for the time you put in.

## What's next for Tetra?

I definitely plan to keep passively maintaining it, for the short term at least - people have built really cool stuff on top of Tetra, and I would feel awful if I just pulled the rug out from underneath them.

Beyond that, I'm not entirely sure! I don't know how I feel about the idea of handing over Tetra to a completely new maintainer; partly because I'd rather people rally around one of the other engines like GGEZ or Macroquad, and partly because I feel quite protective of the work I've done and don't want to see it go off in the wrong direction. I'm going to give this some thought over the next few months - please let me know if you have any strong opinions!

## What's next for me?

I'm hoping that 2022 will be the year I actually make games, rather than spending all my time tinkering! I plan on participating in more game jams, and I want to see if I can turn my RPG prototype into something more substantial.

I don't plan on using Tetra for these. Over the Christmas break I tried using SDL2 and OpenGL (via [`glow`](https://github.com/grovesNL/glow)) directly, and I'm really enjoying that approach so far! The API I've built on top of them looks fairly Tetra-ish, but I'm very deliberately only building what I need, as I need it, to keep myself focused. Maybe one day I will release some of that as open-source, but if so, it'll probably be in the form of smaller libraries rather than a full engine.

I'm also very eagerly awaiting the public beta of the [Luxe](https://luxeengine.com/) game engine, which is supposedly going to be pretty easy to bind to with Rust.

Outside of coding, I also plan on doing a lot more music in 2022. I'm really happy with how [my series of game music covers](https://www.youtube.com/playlist?list=PLbi0QO1oQ88s-ek9uLbrMyabGB60mM8Z7) is turning out, and I really want to throw myself into that again this year - if only to justify the obscene amount I spent on VST plugins on Black Friday...

Thanks for all the support!