---
layout: post
title: "Differentiate Environments with Different Favicons"
date: 2014-02-19 14:04
external-url: http://joelencioni.com/blog/2014/02/19/differentiate-environments-with-different-favicons/
comments: true
sharing: false
author: Joe Lencioni
categories:
  - Efficiency
  - Flow
---

Sally has been tasked with making the commenting feature in her company's
webapp be threaded up to 5 levels instead of only 2 levels. As she makes
adjustments to the code, she collects an unprecedented number of browser tabs.
Some of them point at her company's production site, while others point at her
development environment. In many of them, she enabled "super-user" mode, so she
could more freely play with her app while making her changes.

So she goes on coding and messing with the data for most of the afternoon.
She's about ready to wrap up and head home for the evening when her boss walks
in and asks, "Sally, lots of comments have gone missing. Do you know what
happened?" Sally felt like she had swallowed a sack of doorknobs. In the midst
of her coding spree, she mixed up some of her browser tabs and accidentally
deleted live data. A lot of live data.

<!-- more -->

---

At [Causes], I often have a number of tabs open. Much like Sally, many of these
are pointed at my development environment or the production environment. While
I haven't been as unfortunate as Sally and accidentally done something in
production, I noticed that I spent a fair amount of time searching through my
tabs for the one I was looking for. When pairing with my colleagues, I noticed
that they were spending a good amount of time doing the same thing. Although it
wasn't a serious time commitment by any means, it slowed me down regularly.
Sometimes, it was enough of a speed bump that took me out of my flow. I decided
that there must be a way to improve this situation.

The browser gives you two pieces of information in the tab: the page title and
the favicon. This information should provide you with enough context for you to
make a decision as to which tab you want to foist your full attention.

Since page titles are often the same between environments, that's not a very
useful way to differentiate between these types of tabs out of the box. I
considered adding a prefix to the page title in development, as you might see
with subject lines on email lists, but page titles are often hidden when you
have lots of tabs open. Additionally, page titles feel like page-specific
content, and we typically like things like that to be as similar as possible
between development and production.

Then I realized that we could pack this information into our favicon. The
favicon is the perfect place to put this because they are often indicators of a
domain, not a page. Also, favicons are the last piece of information to be
hidden when opening definitely way too many tabs.

I hit the command line and simply inverted the colors on our production favicon
using ImageMagick, added some configuration to our development environment to
use the inverted image, and I was up and running. Now it is much easier to
figure out which tab is the one I am looking for.

In a Rails environment, for example, you can make this magic happen by first
defining a constant in your application configuration that points to the
location of your production favicon file.

```ruby config/application.rb
FAVICON = 'icons/favicon.png'
```

And in your development environment configuration, override this constant to
point to a different favicon file:

```ruby config/environments/development.rb
FAVICON = 'icons/favicon-inverted.png'
```

Then simply consume the constant in your layouts and you should be good to go.

This quick adjustment is so easy to do and will make you and your team a little
bit faster every single day. It might even prevent some headaches. And, don't
worry about Sally—thankfully she had only soft-deleted the comments.  All of
the data was easily restored.

[Causes]: https://www.causes.com
