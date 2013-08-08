---
layout: post
title: Managing side-effects with the Pub-Sub model
date: 2013-08-08 17:00:00
author: Greg Hurrell
categories:
  - Architecture
  - Maintainability
  - Ruby
---

Over time, large-scale object-oriented systems tend to produce [God
Objects](http://en.wikipedia.org/wiki/God_object). These are classes which know
too much or do too much. They have connections to disparate and varied parts of
the system. They depend on everything, and everything depends on them. They make
systems slow to work with, intractable and hard to modify. They insidiously
undermine and resist our efforts to carry out our core task as engineers:
decomposing complex problems into smaller subproblems that are more easily
solved.

At [Causes](http://www.causes.com) there are some concepts that are
front-and-center in our product: users, their campaigns and the actions
they create to make an impact (things like petitions, fundraisers and pledges).
The concepts are so core that they have a tendency to become God Objects unless
we diligently work to prevent them accruing more and more functionality.

We've lately applied the
[Pub-Sub](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)
(Publish-Subscribe) model to our Ruby code, applying the familiar event-based
patterns that we know from JavaScript to the server side, in an effort to reduce
the tight coupling that some of these God Objects have to other parts of the
system. With a simple, framework-agnostic Ruby library, we've been able to
significantly tame some of the complexity around these classes, and we've
released it as a Ruby gem, [PubSubHub](https://github.com/causes/pubsubhub).

<!-- more -->

## A case study in managing side-effects: taking action

When a user takes action on our site by, say, signing a petition, there is
potentially a slew of side-effects:

  - we persist a record of the signature to a `signatures` table in the
    database, and an `ActionCredit` (effectively a hundreds-of-millions-of-rows
    journal of all action-taking activity on our site)
  - counter-caches tick up
  - stats events are generated and dispatched to one or more tracking systems
  - a recruiter may receive an on-site notification or an email
  - invitations may be marked as accepted (in the case of the recruiter) or
    "indirectly accepted" (in the case of other, multiple inviters)
  - Facebook Request objects may be cleared out
  - feed events are generated and propagated to feeds
  - a custom [Open Graph](https://developers.facebook.com/docs/opengraph/)
    action is published
  - if the action pushed the campaign over a milestone, a milestone event may be
    generated, which itself could result in feed events being propagated, onsite
    notifications, and an email to the campaign organizer
  - if the action is sponsored by a brand, the signature could trigger a
    donation to a nonprofit (which itself would have other side-effects)

And this is only scratching the surface. Having the `Action` class know about
all these collaborators effectively makes it depend on them just as much as they
depend on it, and it places the class squarely within "God Object" territory.

## Using PubSubHub

With `PubSubHub` we have a centralized registry of events, together with the
listeners that wish to be informed of those events. Our `Action` class now just
has to make a call to `PubSubHub.trigger` to let its collaborators know that
something important happened:

```ruby
def take_action(user, recruiter, options = {})
  # core action-taking mechanics go here...

  # and secondary side-effects occur as a result of...
  PubSubHub.trigger :took_action, self, metadata
end
```

This greatly reduces the clutter and makes the separation between core mechanics
and secondary side-effects clear.

Listener registration looks like this:

```ruby
PubSubHub.register(
  took_action: [
    { listener: AlertBanner,                  async: false },
    { listener: FacebookRequest,              async: true  },
    { listener: InvitationManager,            async: false },
    { listener: Profile,                      async: true  },
    { listener: StatsManager,                 async: true  },
    { listener: AnalyticsManager,             async: false },
    { listener: Campaign,                     async: false },
    { listener: NotificationDistributor,      async: false },
    # etc ...
  ],
  # other event types here...
)
```

We considered making listener registration more distributed, via a DSL that
could be sprinkled into other classes, but in the end having the central
registry provides us with a nice inventory of the relationships and dependencies
between different parts of the system. It also conveniently avoids load-order
issues in the context of a large Rails app, whose loading behavior (eager vs
lazy, cached vs uncached) is different between development and production; we
can just stick the registration in an initializer and be done with it.

Note that with this change we've inverted the dependencies such that the
all-important `Action` class no longer depends on a bunch of other classes;
rather, those other classes depend on it. From the perspective of the `Action`
class, this is a good thing: if your goal is to build something that is both
robust and useful, depending on as little as possible and having others depend
on you is a good thing.

One other nicety of this system is that it gives us a straight-forward way to
divide side-effects into the urgent and the non-urgent, the latter being run
asynchronously.

The final piece of the puzzle are the various listeners. By convention, they
implement a handler of the form `handle_<event_name>`:

```ruby
def handle_took_action(action, metadata)
  return unless campaign = action.campaign
  return unless user     = metadata[:user]

  Follow.where(followed_id:   campaign,
               followed_type: Campaign,
               user_id:       user).first_or_create!
end
```

## Next steps

This slender little library has allowed us to scoop out a lot of functionality
from our `Action` class, making it significantly less god-like. New engineers
are able to arrive in the implementation file for the first time and comprehend
the core structure and functionality more rapidly, free from the distraction of
a bunch of secondary and tertiary side-effects.

If you'd like to see what PubSubHub can do for your code base, it's only a `gem
install pubsubhub` away, and the [source
code](https://github.com/causes/pubsubhub) is up on GitHub.

We're mindful that to the person with a hammer, everything looks like a nail, so
we're careful to ensure that we use the tool judiciously. In the context of a
Rails application, this means that we continue to make use of Rails' built-in
tools for managing side effects (things like [Active Record life-cycle
callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html),
[observers](https://github.com/rails/rails-observers), and [Active Support
Notifications](http://api.rubyonrails.org/classes/ActiveSupport/Notifications.html)).

Additionally, our eyes are ever on the prize, asking the question, "How can we
make this simpler?" The Pub-Sub pattern is a tool for loosening the coupling
between parts of the system, but it does not entirely eliminate that coupling.
[Complexity](http://firstround.com/article/The-one-cost-engineers-and-product-managers-dont-consider)
is the ultimate enemy, and the best way to manage side-effects is to simply
eliminate them in the first place.
