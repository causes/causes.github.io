---
layout: post
title: "Showing Color Chips from Sass Variables"
date: 2013-06-06 14:00:00
external-url: http://joelencioni.com/blog/2013/06/06/showing-color-chips-from-sass-variables/
author: Joe Lencioni
categories:
  - Sass
---

At [Causes][0] we like to make our code more maintainable by building reusable
components. Part of this strategy includes [assigning variable names to hex
colors using Sass][1]. This allows us to more easily reuse the same colors
everywhere, which improves consistency and makes it easier to re-color the
entire site when our design needs change.

To increase the visibility of these reusable components, we've been building
out a collection of things that designers and engineers can reference, drop in
to projects, and iterate upon. The code sits next to the rendered version,
allowing people to easily see the implementation required to produce the
result. We call this collection the component gallery. It helps us do more with
less code, be more consistent, and iterate on global changes more easily and
effectively.

When we started fleshing out the color variables we wanted to use throughout
the site, it seemed natural to show these colors in the component gallery as
Pantone color chips. That way, designers could reference the colors that we are
using, we'd have a single place to see that all of the colors look great next
to each other, and engineers could easily pluck variable names when
implementing designs to match the mocks.

{% img /assets/2013-06-06-pantone-chips.png %}

<!-- more -->

The quick solution would have had us defining these colors in two places: once
in Sass, where we actually need to use them, and again in Ruby, where we want
to render them on a page. However, we sensed that this would be a pain to
maintain and the two different definitions would quickly fall out of sync.

To resolve these issues, we decided to move our color variables into their own
Sass partial, and use Sass to parse it to display the colors and their
variables. Since we like to use [functions like `darken()`, `lighten()`, and
`scale-color()`][2], we needed a solution that would actually execute the Sass
and use the result.

So we built a [Sass Visitor][3] to evaluate variables and return pairs of names
and values:

```ruby
class SassVariableEvaluator < Sass::Tree::Visitors::Base
  def visit_comment(node)
    # prevents empty arrays from being in the returned array
  end

  def visit_variable(node)
    @environment ||= Sass::Environment.new
    @environment.set_local_var(node.name, node.expr)
    [node.name, node.expr.perform(@environment)]
  end
end
```

Then we have our `SassVariableEvaluator` visitor evaluate the variables in our
colors Sass partial:

```ruby
def color_variables
  engine = Sass::Engine.for_file('path/to/_colors.css.scss', syntax: :scss)
  SassVariableEvaluator.visit(engine.to_tree).compact
end
```

Looping over those evaluated variables in our view generates the desired
markup:

```haml
.clearfix
  - color_variables.each do |name, value|
    .pantone
      .chip{ style: "background-color: #{value};" }
      %var $#{name}
```

And style them nicely:

```sass
.pantone {
  @include box-shadow(0 0 5px rgba(0, 0, 0, .1));
  border: 1px solid #eee;
  float: left;
  font-size: 11px;
  margin: 0 1em 1em 0;
  width: 145px;

  &:nth-child(4n) {
    margin-right: 0;
  }

  .chip {
    background: #f00;
    height: 145px;
  }

  var {
    display: block;
    font-style: normal;
    font-weight: bold;
    height: 40px;
    padding: .75em;
  }
}
```

[0]: http://www.causes.com
[1]: http://joelencioni.com/blog/2013/03/16/10-easy-ways-to-craft-more-readable-css/#two-or-more-use-a-for
[2]: http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html
[3]: http://sass-lang.com/docs/yardoc/Sass/Tree/Visitors/Base.html
