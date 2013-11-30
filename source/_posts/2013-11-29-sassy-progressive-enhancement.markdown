---
layout: post
title: "Sassy Progressive Enhancement"
date: 2013-11-29 15:50
external-url: http://joelencioni.com/blog/2013/11/29/sassy-progressive-enhancement/
comments: false
sharing: false
author: Joe Lencioni
categories:
  - Sass
  - Media queries
  - Progressive enhancement
---

A mobile-first approach to design can be good: it helps you focus and
potentially simplify your requirements, which can translate into a better user
experience. Likewise, a mobile-first approach to implementation can be good: it
puts the more expensive tasks on the shoulders of clients that are more likely
to be able to handle the extra load, which can also translate to a better user
experience.

At [Causes], we have been building things with a mobile-first approach both
in terms of design and implementation with a goal of progressive enhancement.
Since switching to this mode on the technical side, we have noticed that our
code is easier to write and more coherent to read, both of which are big
benefits for the engineering team.

So we've put together a little project that helps us write readable media
queries in this way. We call it [sass-enhance].

<!-- more -->

## sass-enhance In Action

sass-enhance defines mixins for media queries, `enhance` and `degrade`.

These mixins each take a breakpoint as an argument, and a block of styles to
apply when that breakpoint is activated. Optionally, you can specify ranged
breakpoints using `until`.

Breakpoints can be named, as defined in the `$breakpoint-max-widths` variable
(e.g. "tablet"), or arbitrary widths (e.g. "720px").

### `enhance()`

The `enhance` mixin is used to apply styles to a selector as the viewport gets
wider. It can be used to progressively enhance a page. We prefer using
`enhance` over `degrade` because it is a mobile-first implementation that tends
to be simpler in its execution.

To adjust the padding from 1em to 2em at the desktop breakpoint and wider, you
could use the following SCSS:

```scss
.my-selector {
  padding: 1em;

  @include enhance(desktop) {
    padding: 2em;
  }
}
```

Which compiles to:

```css
.my-selector {
  padding: 1em;
}

@media screen and (min-width: 960px) {
  .my-selector {
    padding: 2em;
  }
}
```

If you wanted to only apply a different amount of padding for only the tablet
viewport width and nothing wider or narrower, you could use the following SCSS:

```scss
.my-selector {
  padding: 1em;

  @include enhance(tablet until desktop) {
    padding: 2em;
  }
}
```

Which compiles to:

```css
.my-selector {
  padding: 1em;
}

@media screen and (min-width: 641px) and (max-width: 959px) {
  .my-selector {
    padding: 2em;
  }
}
```

### `degrade()`

There are some cases where `enhance` does not work or make sense. In these
cases, it is okay to use `degrade` to gracefully degrade the styles as the
viewport gets narrower.

To adjust the padding from 2em to 1em at the tablet breakpoint and narrower,
you could use the following SCSS:

```scss
.my-selector {
  padding: 2em;

  @include degrade(tablet) {
    padding: 1em;
  }
}
```

Note: this produces functionally equivalent styles as the first example.

Likewise, if you wanted to only apply a different amount of padding for only
the tablet viewport width and nothing wider or narrower, you could use the
following SCSS:

```scss
.my-selector {
  padding: 2em;

  @include degrade(tablet until small-tablet) {
    padding: 1em;
  }
}
```

## Installation

The recommended way to get set up with sass-enhance is via [Bower], the
front-end package manager. To do this, simply run:

```bash
bower install sass-enhance
```

If your project uses Bower to manage its dependencies and you'd like to save
sass-enhance as a dependency in your project's `bower.json` manifest file, add
the `--save` option:

```bash
bower install --save sass-enhance
```

Once you have the files locally, you simply need to import it so it is
available to your stylesheets. At Causes, we are building a Rails app, so we
symlink sass-enhance into `vendor/assets/stylesheets/sass-enhance/` and
`@import` it ear the top of our SCSS files:

```scss
@import 'sass-enhance/sass-enhance';
```

## Configuration

Although sass-enhance will work just fine right out of the box, you may be
itching to configure it to fit your needs just right. For this purpose, we have
included a variable that you can set, called `$breakpoint-max-widths`. Just
make sure to set this variable *before importing sass-enhance* or your settings
will not work.

This variable is a comma separated list of breakpoint names and max-width
pairs. You can choose whatever names and widths you prefer. The default is
something like:

```scss
$breakpoint-max-widths: mobile           360px,
                        mobile-landscape 500px,
                        small-tablet     640px,
                        tablet           959px,
                        desktop          99999px;
```

So, if you prefer media queries with more generic names, you are free to
configure it to suit your needs:

```scss
$breakpoint-max-widths: small  400px,
                        medium 800px,
                        large  1000px;
```

## Conclusion

We hope that you find this tool useful. Pull requests are welcome. [View the
source on GitHub][sass-enhance].

[Causes]: https://www.causes.com
[sass-enhance]: https://github.com/causes/sass-enhance
[Bower]: http://bower.io
