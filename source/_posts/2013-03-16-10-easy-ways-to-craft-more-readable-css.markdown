---
layout: post
title: 10 Easy Ways to Craft More Readable CSS
date: 2013-03-16 15:45
external-url: http://joelencioni.com/blog/2013/03/16/10-easy-ways-to-craft-more-readable-css/
author: Joe Lencioni
categories:
  - CSS
  - Readability
  - Sass
  - Stylesheets
---

> Always code as if the [person] who ends up maintaining your code will be a
> violent psychopath who knows where you live. Code for readability.
> —<cite>[John Woods](https://groups.google.com/d/msg/comp.lang.c++/rYCO5yn4lXw/oITtSkZOtoUJ)</cite>

Diving into a large, old piece of CSS typically is neither easy nor
pleasurable. I find that **the biggest challenges in working with old CSS often
lie in understanding the purpose and interactions of the styles**.

When styling new elements, we have the entire context of the implementation
immediately available, and it is easy to write styles that make sense to us at
that very moment. However, in a few weeks or to a fresh pair of eyes, what made
a lot of sense at first might end up being a lot more cryptic. Without a clear
understanding of the purpose and interactions of the styles, modifying
stylesheets can be dangerous, tedious, and cumbersome. Therefore, it is
important to communicate enough context so that future developers will be able
to grok the code easily and make informed decisions.

At [Causes](http://www.causes.com/), we have adopted the following practices
which we believe have improved the maintainability of our stylesheets, reduced
bugs, and increased developer velocity. When you have finished reading this, I
hope that you will have a few more tools to help move your codebase toward
greater maintainability.

[Read on &rarr;](http://joelencioni.com/blog/2013/03/16/10-easy-ways-to-craft-more-readable-css/)
