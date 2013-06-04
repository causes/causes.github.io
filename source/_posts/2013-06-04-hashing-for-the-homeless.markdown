---
author: Jeremy Dunck
layout: post
title: 'Hashing for the homeless'
date: 2013-06-04 13:57
comments: true
categories:
  - Activism
  - Homelessness
  - ToolsOfChange
---

The <cite>[Legend of John
Henry](http://en.wikipedia.org/wiki/John_Henry_%28folklore%29)</cite> holds a steel
driver in contest with a steam-powered hammer, casting human excellence against
the progress of technology. Henry is a folk hero, but he dies betting against
technology. The point isn't that technology wins, though — we don't really
lose when technology wins.

{% img left /assets/2013-06-04-john-henry.jpg 576 480 John Henry %}

<iframe width="400" height="80" src="https://rd.io/i/QFdyK3YSgg/"
frameborder="0"></iframe>

I'm a technologist, but I'm not in it for the tech — I'm in it for making
things that make things better. Code is an unreasonably effective lever. It
sounds syrupy to say it, but I believe that computers are the best way for me
to make the world better. They work faster and more tirelessly than I could.
Bits travel the world freely, providing value to people I will never meet.
Things I wrote 10 years ago are still doing useful work, and they may continue
to do so after I am gone.

But there is a gap in society. Technologists are seen as wizards, in the
Matrix. Sometimes they are shown as heroic, nerdy, or villainous, but always
unassailably "other". Normal people don't do this thing. Normal people feel
disenfranchised by technology. Some people feel it's useful but don't see
themselves ever producing with it. Very few people see technology for what it
really is: a tool for your use. People often suppose that tech sophistication
is a function of generation — that there was a web generation, a mobile
generation, that the next generation will get it better.

I disagree. Tech is changing more rapidly, not less, while our ability to
incorporate the new capabilities into our practices, norms, and laws is staying
constant. Each generation's youth get a head start on incorporating new things
because early in life everything is new — we have [fewer bad patterns to
match](http://en.wikipedia.org/wiki/List_of_cognitive_biases). But the faster
the strides of tech, the more quickly youth's head start is overtaken. But we
can choose not to be John Henry.

It would help everybody if we worked to close this gap, and casting the gap as
generational does real harm because it encourages waiting while we might act.

<!-- more -->
The computer and the internet — they carry meaning only in relation to our
connections, interactions, and outcomes. (Picasso quipped that computers are
useless because they can only give you answers.)

I've [written
before](http://anabasis.dunck.us/2012/11/25/should-everybody-be-a-programmer/)
that everyone should be a programmer; but if the computer is already a tool for
your use, please think back about how you got to this point. Why does it
matter to you, and what might have changed that for you? How can you make that
change for the people in your life? How can you apply this rare talent — of
being a wizard — to things that make material differences in people's lives?

I've had some success appealing through automation. Tell your friends that if
they've ever been bored while using a computer, they've just found a reason to
learn programming. Most people who have used a computer have done some tedious
work on it — categorizing pictures, or filling in spreadsheets, for example.
Using that time to learn and to automate is [probably
worthwhile](http://xkcd.com/1205/).

But it turns out that hashing is also helpful to the homeless.

I recently met with [Doniece Sandoval](https://twitter.com/LavaMae) of [Lava
Mae](http://www.lavamae.org/). She's working on providing mobile showers to
homeless people here in San Francisco. If you've ever been to San Francisco,
you've seen and smelled [how many homeless people there
are](http://en.wikipedia.org/wiki/Homelessness_in_the_United_States). These
people lack basic facilities, and there's a tremendous shortage of shelters and
showers. What few facilities there are are in a concentrated area of the city,
while the homeless spread throughout the city. Showering even once a month is
challenging. Mobile showers are an interesting approach to this problem,
because they are more scalable than fixed facilities, and because they can
bring the service to people who have precious few travel options and resources.

Doniece has a problem, though — she needs homeless people to know when her
service will be available — what time of day and where in the city. But not
only that — she has [NIMBY](http://en.wikipedia.org/wiki/NIMBY) to address.
Nobody wants a huge line of homeless people loitering around their neighborhood
in hopes of a shower. Lines will form because her service will be overwhelmed
with demand.

She was exploring options to address these opposing concerns. Perhaps
unannounced popup locations for fixed amounts of time, too short for lines to
form? Pretty unfair and difficult logistically. Perhaps homeless people could
sign up to the service, receive a wristband, and be notified when their number
came up? Pretty expensive and hard to manage.

It occurred to me that she could use a [hashing
algorithm](http://en.wikipedia.org/wiki/Hash_function). Lava Mae could always
start service at the same time each day, and a location per day. People could
be assigned time slots during the day based on something they already have: a
name. The exact time slotting doesn't matter - it could be as simple as 10
minutes per letter, so that people whose names start with A come at 9AM, and
names starting with D come at 9:30, and so on. It's important that the hash be
simple and intuitive, but the direct mapping of letter to a specific time isn't
the only approach. Another could be that A through D collectively get the
first hour, or whatever other grouping/distribution is useful.

{% img left /assets/2013-06-04-hash.png 240 184 Hash function %}

With this approach, everybody knows when they can get a shower (it stays the
same time for each person), and there's no point coming earlier or staying
later. Checking whether people are allowed at that time is simple — any form
of ID would do — and it would be fair (assuming time slots were adjusted to
per-letter population). If there are too many people for a given time slot, at
least each person in that slot got a fair chance (everybody who came for a time
slot could get picked in a small lottery). NIMBY and wasted time are both
minimized.

To a programmer, this idea is quite simple — we use hashing all the time,
mapping arbitrary values into a known space. We think abstractly about solving
problems all the time. But it solves a real-world problem in a practical way.
Any time a service needs to be fairly distributed from a thing "they" have to a
thing we control, hashing probably will help.

Activism can be, in part, an engineering problem. And we can all be engineers.
Maybe you should be an activist. Or maybe you should help an activist become
an engineer. [Causes is
hiring](http://www.causes.com/jobs?ctm=engblog#Engineering) if you'd like to
get started here.
