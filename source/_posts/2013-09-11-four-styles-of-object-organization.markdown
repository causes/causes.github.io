---
layout: post
title: "Four Styles of Function Organization"
date: 2013-09-11 13:48
author: Elliot Block
categories:
  - Architecture
  - Maintainability
  - Ruby
---

In object-oriented code, there are several choices for where to put a
new function, and each choice has its pros and cons.  These choices
repeat themselves over and over in a codebase, so it's worth reviewing
the tradeoffs we make on a daily basis.

<!-- more -->

## Methods directly on core models

In <abbr title="Object Oriented Programming">OOP</abbr> frameworks
like Rails, a method's default home is the class that holds its state.
Suppose that the `User` class has Facebook friends, Twitter followers,
and LinkedIn connections.  The simplest approach is:

```ruby
class User
  def facebook_friends; ...; end
  def twitter_followers; ...; end
  def linkedin_connections; ...; end
end

# invocation is:
user.facebook_friends()
```

The pros of this arrangement are its simplicity and obviousness.  All of
the methods on `User` can be found explicitly listed in the `user.rb`
file.

The problem with putting everything in one model is that eventually it
contains many inessential features.  The pathological case looks like:

```ruby
class User
  def facebook_method0; ...; end
  def facebook_method1; ...; end
  def facebook_method2; ...; end
  def twitter_method0; ...; end
  def twitter_method1; ...; end
  def twitter_method2; ...; end
  def linkedin_method0; ...; end
  def linkedin_method1; ...; end
  def linkedin_method2; ...; end
end
```

This is the **obese model** problem.  Too much code in one model and
too little organization makes a monolithic `User` class a poor unit of
organization.  An obese class is hard to understand because it contains
too much unstructured code inside it.  Consumers of obese classes are
also harder to understand, because the dependency between the consumer
and the consumed is defined in terms of a large, imprecise concept (the
obese model), rather than a small and precise one.

Putting methods directly into core models is a simple, direct and
default solution that works well for small situations.  By the
[YAGNI](http://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)
principle, it is probably still the best place to start.  However for
large projects, the downsides of this organization start to show.

In Ruby-land, a proliferation of concerns inside an object often leads
to a second style of code organization: mixins.

## Mixins

Ruby supports mixins as a form of code reuse and concise organization.
Mixins declare code separately and then include them directly into
another class.  For example:

```ruby
module Facebook
  def facebook_method0; ...; end
  def facebook_method1; ...; end
  def facebook_method2; ...; end
end

class User
  include Facebook
end

# invoking:
user.facebook_method0()
```

Mixins can improve both organization and code reuse.  In this example,
the Facebook-related methods are grouped together in one place, and
separated out from `User`.  The `Facebook` module can then be mixed into
a variety of other classes.  In this organization, the pros are that the
related methods are grouped into one place (the `Facebook` module),
keeping both the individual concern and the target class (`User`)
cleaner.

The first problem with mixins is that `User` objects now have methods
that cannot be found directly in `User`.  In order to figure out how
`User` got `facebook_method0`, and what `facebook_method0` does, you
can't see it by looking at `User`; you have to realize it's getting
`include`d, usually by grepping the entire codebase for
`facebook_method0` [&#91;1&#93;](#notes), which is unfortunate.  This
dislocation also increases the chances of method name collision
between the mixin and the host class.

A second problem with mixins is that they tend to have mysterious and
implicit dependencies upon their host classes, especially if they were
first written coupled-into the host, and then extracted without
generalizing.  Mixins used only once are particularly prone to this
"separation is not organization" problem.

From the outside, the mixin solution looks structurally identical to
just putting all the methods in the object: you see an object with a
ton of methods on it.  Thus mixin solutions share basic properties
with a large model, but with a tradeoff:

- Better organization of a single concern into a single place
- Some code reusability
- Worse clarity on where methods are coming from
- Often worse clarity on code flow, since mixin methods are interacting
  with base class methods, and you'll find yourself ping-ponging between
  reading the two files.

Mixins can make code more concise, and somewhat better organized, but
often at the cost of the code being much less clear.

Mixins often prompt people to turn to _delegation_.

## Delegation

Because mixins are sometimes frustratingly invisible, we also see
people take an opposing approach: explicitly delegating functionality
to underlying objects.  The separately concerned functionality goes
into its own class/module.  For example:

```ruby
module Facebook
  def self.method0(user); ...; end
  def self.method1(user); ...; end
  def self.method2(user); ...; end
end

# basic delegation
class User
  def facebook_method0
    Facebook.method0(self)
  end

  def facebook_method1
    Facebook.method1(self)
  end

  def facebook_method2
    Facebook.method2(self)
  end
end

# invoking:
user.facebook_method0()
```

The strength of this organization is that the individual `Facebook`
concern is clearly organized into its own module, and explicitly
called from `User`.  This improves organization, reusability, and
simplicity of the execution path.  One downside is that `User` is back
to having every possible method within it, though most of them are
mere portals to the separate module.

From the outside the `User` class still looks the same: it has a ton of
methods on it.  So the cons for this style of organization remain mostly
the same as the original "shove all the methods in the class."  On the
plus side, there is slightly less complexity inside `User` itself,
because some has been separated out into `Facebook`.

This is a verbose but somewhat clearer organization of the code--if
you like delegation, there are several slightly less verbose
approaches available [&#91;2&#93;](#notes).  But delegation's
verbosity and continued coupling of everything into `User` still call
out for a better alternative.

## Namespaces

The persistent problem above is dealing with peripheral functionality
such as `Facebook` coupled into core functionality such as `User`.  Even
if internally a class has been cleaned up to only delegate methods, the
external complexity of the class has not improved: consumers of `User`
still see a sprawling and complex interface of functions.

We can clean up massive classes to have smaller and more specialized
interfaces, by splitting up concerns into namespaces like this:

```ruby
module Facebook
  def self.method0(user); ...; end
  def self.method0(user); ...; end
  def self.method0(user); ...; end
end

class User
end

# invoking
Facebook.method0(user)
```

The most important part of this arrangement is that instead of methods
being inside `User`, they're called from the outside, from `Facebook`,
which takes a user.

The pros of this organization are that the separate concern (of
Facebook, of Twitter, etc.) has finally truly been extracted out of
`User`.  There is no crowding inside `User`, no implicit method
declarations, and all of the code for Facebook is inside the `Facebook`
module.

This addresses most of the original concerns.  The disadvantage in
this situation are that this could eventually proliferate into many
fragmented modules, and that there is no longer a single home for for
user-related method declarations.

Using namespaces has a variety of structural advantages, which we'll
treat in a different article.  For now we hope to have shown that,
strictly for basic comprehension, there are tradeoffs worth
considering for consolidating vs. separating code.

## Conclusion

Code organization choices are deceptively mundane: they seem like
small decisions, but have compounding long term impacts on the
malleability of our code.  Because we think and talk in terms of code,
clear code help us understand clearly, move quickly and reason
correctly.  This makes organization a fundamental investment we can
make in our software.

## Notes
<a name="notes"></a>

&#91;1&#93; You can also find a method definition via

    my_object.method(:my_method_name).source_location

&#91;2&#93; There are terser delegation patterns; Rails comes with a
  `delegate` method, and the Ruby standard lib has a `Delegator`
  class, a `SimpleDelegator` class, a `Forwardable` module, which can
  all be used for various styles of delegation.

Special thanks to Andrew Berls, Jeremy Dunck, Preston Guillory, Greg
Hurrell,  Joe Lencioni, and Nebs Petrovic for their thoughts.
