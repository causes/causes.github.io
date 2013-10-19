---
layout: post
title: "Why Test Performance Matters"
date: 2013-11-06 17:50
author: Henric Trotzig
categories:
 - Testing
 - Performance
---
"How fast are your tests?".  Have you ever had that question? Probably not.
Does performance matter for tests? Yes, and here's why: The time it takes from
you changing a bit of code until you get feedback on whether or not that code
works is crucial in knowing if your code is good or not. Let me illustrate by
two scenarios.

<!-- more -->

### Scenario 1: Slow feedback cycle
Let's say your tests take 20 minutes to run. Some time ago developers got tired
of running the tests locally so you moved to a model where you run the tests in
30 minute intervals on your build server. Now you have just commited some code
that changes the default sort order for getting lists of users from the
database. You commit and push your code and start working on something else. An
hour later you get an email: "Build failed". You quickly scan through the email
to see that 7 commits have been pushed since the last build and that this test
is failing: "main_menu_has_correct_items". That doesn't look like your fault
(right?) so you continue working on that other thing. Meanwhile, two other
contributors to this build are scratching their heads for hours trying to
figure out why the menu isn't rendering correctly, only to find out that the
sort order for users was making the test fail.

### Scenario 2: Rapid feedback cycle
Your tests are lightning fast. Every time you save your code, tests will be run
automatically and tell you whether or not you broke something. You change the
sort order for getting lists of users from the database and notice within
seconds that the 'main_menu_has_correct_items' test is failing. "Strange", you
think and revert your change just to make sure that it wasn't a fluke. Nope, no
fluke. The test is now passing. You make your change again and start
investigating why changing the user sort order had an effect on the main menu.
You quickly find that the menu was depending on the last item from a list of
users.

## Speed matters
The above scenarios should make it obvious that performance matters for tests.
The examples are taken to the extremes, and you will probably find that your
company is somewhere in the middle in this spectrum. You might not reach
instantaneous feedback as in scenario 2, but you can certainly aim to get
closer to that. At [Causes](https://www.causes.com), we are down to seconds in
most scenarios. Our entire test suite takes about 5 minutes to run, but clever
engineers have created a smart tool called `relevant-tests` that tells you with
high confidence what tests you need to run, and you can run only those tests in
seconds through the help of [Zeus](https://github.com/burke/zeus), a
Rails daemon that speeds up most Rails-related tasks by keeping a runtime
environment hot at all times, and cloning that environment when you need it.
