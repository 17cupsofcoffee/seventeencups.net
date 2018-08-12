+++
title = "Hello, Gutenberg!"
date = 2017-10-01

[taxonomies]
tags = ["rust", "website"]
+++

In [my first post on this site](./posts/finally-up-and-running.md), I mentioned that I'd tried several static site generators and found them all to be fairly painful to use - hence my choice to use [Ghost](https://ghost.org/). While I still think Ghost is a nice platform, there's a few things that made me rethink that decision recently:

* I was paying $5 a month for VPS hosting, which is cheap enough, but seems like a waste when I could just put a static site on Netlify for free.
* Updating Ghost is a fairly manual process, and I kept forgetting to do it.

So, I went back on the hunt. [Gatsby](gatsbyjs.org/), a React-based site generator, caught my interest, but I found it to be far too complex for my basic needs. I just want to render Markdown - why do I need three plugins and a load of Node glue code?! That said, if I wanted to interact with a headless CMS I could see it being a good choice, and I might revist it it later down the line.

[Hugo](http://gohugo.io), on the other hand, came extremely close to winning me over (and would probably still be my second choice). It has loads of features out of the box, and has incredibly fast build times. Unfortunately, Go templates are fairly nasty to work with, and I had a few issues getting the new syntax highlighting engine working.

Then I came across [Gutenberg](https://github.com/Keats/gutenberg), by [Vincent Prouillet](https://vincent.is/). It has a lot of the nice features that attracted me to Hugo, plus a few big advantages:

* It uses the [Tera](https://github.com/Keats/tera) templating language (also written by Mr. Prouillet!), which is quite nice and has a lot in common with Django and Jinja.
* It has Sass built in, which removes a lot of the need for a frontend build pipeline.
* It's written in Rust, meaning if I run into any issues with it, I can actually contribute rather than waiting for someone else to fix it!

After spending the weekend moving my site across to Gutenberg, I'm really happy with it - there's a few bugs, and the documentation is a little scattered, but on the whole, it just feels really simple and pleasant to use. The majority of static site generators I've tried, I've felt like I was battling them to get simple stuff done, but Gutenberg kinda just got out of my way and let me get on with it.

While I was at it, I gave the site a bit of a spruce up - I think it has a little more personality now! Let me know what you think.
