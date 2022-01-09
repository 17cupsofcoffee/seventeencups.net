+++
title = "Simple HTTPS Domain Redirects with Netlify"
description = "Tips on using (or abusing?) the Netlify _redirects file for free HTTPS domain redirection."
date = 2020-06-19

[taxonomies]
tags = ["web"]
+++

This week, a website that I help maintain ran into some issues with their domain names. I stumbled upon a quick (and free!) solution which I thought was worth sharing, in case it helps anyone else out of a jam.

**TL;DR:** You can use a Netlify [`_redirects`](https://docs.netlify.com/routing/redirects) file to redirect both HTTP and HTTPS traffic from one domain to another, without paying for server hosting or an SSL certificate.

## The Problem

[Are We Game Yet?](https://arewegameyet.rs/) is a website which lists resources for game developers in the Rust community. It's hosted on Github Pages, and historically it's been available via `https://arewegameyet.com`. We also had simple HTTP redirects set up on `http://arewegameyet.rs` and `http://arewegameyet.org`. 

Since the `.rs` domain makes it a bit clearer that the website is Rust-related, we decided that it'd be good to make this the new primary URL. But that led us to a bit of a conundrum: pretty much all of the existing links to the site, including the ones on Google, point at `https://arewegameyet.com`.

GitHub Pages doesn't support multiple domains, and our domain providers only support simple HTTP redirects unless we pay through the nose for a VPS and/or a SSL certificate. So for a while, it looked like we wouldn't be able fix these URLs without moving our hosting or increasing our expenditure, neither of which would be ideal.

## The Solution

[Netlify](https://www.netlify.com/) is a web hosting service that's targeted at static sites. They have an extremely generous free tier, which I'm currently using to host both this blog and [Tetra's documentation](https://tetra.seventeencups.net/), and I'll basically tell anyone who'll listen about how they're the best thing since sliced bread.

One of the really cool features of Netlify is that they allow you to specify [redirect rules](https://docs.netlify.com/routing/redirects/#syntax-for-the-redirects-file) as part of your static deployment. You provide a file called `_redirects`, and their servers will handle mapping the URLs. It's similar to an Apache `.htaccess` file, but without having to actually manage the server itself.

I've seen a lot of cool tricks done with `_redirects` files before; for example, [Kent C. Dodds](https://kentcdodds.com/) has used them to create his own [URL shortener](https://github.com/kentcdodds/netlify-shortener). So I was curious whether they could be applied to this problem - and as it turned out, thanks to [splat redirects](https://docs.netlify.com/routing/redirects/redirect-options/#splats), they can!

First, we created a dummy `index.html` file, as Netlify didn't seem to want to deploy a site without one present. Then, alongside it, we created a `_redirects` file with only two lines:

```
/* https://arewegameyet.rs/:splat 301!
/  https://arewegameyet.rs        301!
```

The `!` at the end is important, as it prevents files in the deployment from [shadowing](https://docs.netlify.com/routing/redirects/rewrites-proxies/#shadowing) the redirects.

We deployed these two files to Netlify, added our `.com` and `.org` domain names to the project, and activated HTTPS via Netlify's [Let's Encrypt](https://letsencrypt.org/) integration.

And that's it! Now both domains redirect all of their traffic on both HTTP and HTTPS to the corresponding page on `https://arewegameyet.rs`, and we didn't have to pay a penny.

Hopefully this is useful to other people who find themselves in a similar situation!