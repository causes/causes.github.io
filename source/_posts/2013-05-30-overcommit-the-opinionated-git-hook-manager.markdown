---
layout: post
title: "Overcommit: The opinionated Git hook manager"
date: 2013-05-30 14:02
author: Aiden Scandella
categories:
  - Git
  - Automation
---

At [Causes](http://www.causes.com), we care deeply about code quality.  We
promote thorough, offline code review through
[Gerrit](https://code.google.com/p/gerrit/) and take pride in each commit we
make. Due to the sheer volume of code review and the number of engineers on our
team, it's important that by the time other engineers review our code we have an
established baseline of quality.

There are a few important ingredients to making a good commit:

  - Correctness: The code does what you expect it to do
  - Commit message: Tim Pope provides an excellent summary of [what makes a good
    commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
  - Style: The code matches our team's coding styles
  - Test coverage: relevant tests have been run, and any new features have spec
    coverage

Enter [overcommit](https://github.com/causes/overcommit). This evolved from a
single file linter into a full-fledged, extensible hook architecture.  Available
as a Ruby gem:

    gem install overcommit

What does it do? In short, it automates away all the tedium before a commit
reaches code review. It ships with a set of opinionated lints that ensure a
level of consistency and quality in our commits.

## In Action

Here's an example of overcommit saving me from committing janky code:

<pre><code>❯❯❯ echo "eval('alert(\"hello world\")');" > eval.js
❯❯❯ git add eval.js
❯❯❯ git commit
Running pre_commit checks
  Checking causes_email...........<span class="success">OK</span>
  Checking test_history...........<span class="warning">No relevant tests for this change...write some?</span>
  Checking restricted_paths.......<span class="success">OK</span>
  Checking js_console_log.........<span class="success">OK</span>
  Checking js_syntax..............<span class="error">FAILED</span>
    eval.js: line 1, col 1, eval can be harmful.

    1 error
  Checking author_name............<span class="success">OK</span>
  Checking whitespace.............<span class="success">OK</span>

<span class="error">!!! One or more pre_commit checks failed</span>
</code></pre>

<!-- more -->

## Installation

After installing the gem, a new binary, `overcommit`, will be available. You can
use this binary to install git hooks into an existing repository like so:

    overcommit my-project

Where `my-project` is the directory of a git repository. In addition to
installing hooks into `my-project/.git/hooks`, this will also write an
`overcommit.yml` file containing repository-specific configuration. You can use
this, for example, to always skip a certain type of lint. See more options by
running `overcommit --help`.

## Built-in functionality

#### `pre-commit` hooks

- `coffee_lint` uses [CoffeeLint](http://www.coffeelint.org/) to keep your
  CoffeeScript clean and consistent.

- `haml_syntax` verifies that any [Haml](http://haml.info/) file to be committed
  is syntactially valid.

- `js_syntax` uses [jshint](http://www.jshint.com/) to ensure best JavaScript
  practices are followed.

- `python_flake8` uses [flake8](https://pypi.python.org/pypi/flake8) to lint
  Python code.

- `ruby_syntax` makes sure that any Ruby files to be committed are syntactically
  valid by  running `ruby -c #{staged_file}`.

- `scss_lint` integrates with our [scss-lint](github.com/causes/scss-lint) gem
  to ensure our stylesheets are the best they can be. This includes alphabetizing
  properties, removing unnecessary units, and so much more. [Read
  more](https://github.com/causes/scss-lint#what-gets-linted) about what gets
  linted.

- `whitespace` verifies that no hard tabs are used and that no trailing
  whitespace is included. The devil is in the details.

There are more hooks included, including Causes-specific ones (such as making
sure your @causes.com email address is used), but these are excluded by
default. See the rest of the lints
[here](https://github.com/causes/overcommit/tree/master/lib/overcommit/plugins/pre_commit).

#### `commit-msg` hooks

- `russian_novel` is just for fun, to reward developers for writing exemplary
  (long) commit messages.

- `text_width` ensures that the body of the commit is hard-wrapped to 72
  characters, and that the subject is <= 60 characters.

- `trailing_period` warns the author if their commit message subject ends with a period.

## Extensibility

In addition to the gem-provided, global hooks, `overcommit` also allows for
repository-specific hooks to be added. An example from the documentation is our
[Chef](http://www.opscode.com/chef/) repository-specific hook to run
[Foodcritic](http://acrmp.github.io/foodcritic/) against any cookbooks being
committed.

This file lives in kitchen.git/.githooks/pre_commit/food_critic.rb

```ruby food_critic.rb
module Overcommit::GitHook
  class FoodCritic < HookSpecificCheck
    include HookRegistry
    COOKBOOKS = 'cookbooks'
    @@options = { :tags => %w[~readme ~fc001] }

    def run_check
      begin
        require 'foodcritic'
      rescue LoadError
        return :stop, 'run `bundle install` to install the foodcritic gem'
      end

      changed_cookbooks = modified_files.map do |file|
        file.split('/')[0..1].join('/') if file.start_with? COOKBOOKS
      end.compact.uniq

      linter = ::FoodCritic::Linter.new
      review = linter.check(changed_cookbooks, @@options)
      return (review.warnings.any? ? :bad : :good), review
    end
  end
end
```

## Skipping checks

There are, of course, times when you're going to need to break the rules. You
can skip any lint by passing the underscore-ized name into the `SKIP_CHECKS`
environment variable. This can either be a single lint, or a
comma/colon-separated list of lints to skip:

    SKIP_CHECKS=js_syntax:restricted_paths git commit

There's also the special value of `all`, which will skip all of the non-required
lints.

## The future

Until recently, overcommit has only been useful inside Causes due to the
specific lints we run and the lack of easy installation. We hope others find it
useful, and have released it under the MIT license. Pull requests are welcome.
[View the source on GitHub](https://github.com/causes/overcommit).

Happy hacking.
