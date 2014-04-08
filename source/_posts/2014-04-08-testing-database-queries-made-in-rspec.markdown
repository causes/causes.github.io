---
layout: post
title: "Testing database queries made in RSpec"
date: 2014-04-08 14:35
author: Henric Trotzig
comments: false
categories:
  - performance
  - Open Source
---

Recently, [Joe] and I have been doing some performance work on [causes.com] to
speed up the time it takes to generate a page on the server. Most of that work
meant reducing the number of database queries we make. Take the [profile page]
for example. On that page, we list the most important campaigns you've
participated in. Before we started optimizing it, we were making more than 10
queries per campaign listed. We added some pre-loading and caching and
managed to take all those database queries down by 90%. That in turn brought
the time it took to generate this page down from about a second to 200
milliseconds.

<!-- more -->

## Testing the avoidance of database queries

In order to verify that the work we did had the effect we intended, and to
guard us against future regressions, we built a tool to use in our unit tests
that counts the number of database queries made by running a particular piece
of code. Here's an example:

```ruby
describe 'rendering a collection of items' do
  let(:items) { Item.last(10) }
  subject { render_items(items) }

  context 'when the items have not been pre-loaded' do
    it 'performs database queries' do
      expect { subject }.to make_database_queries
    end
  end

  context 'when the items have been pre-loaded' do
    before { preload_items(items) }

    it 'does not perform any database queries' do
      expect { subject }.to_not make_database_queries
    end
  end
end
```

The interesting part here is the `make_database_queries` [RSpec] matcher. It
hooks into [ActiveRecord] and detects all database queries made during the
execution of the block. If you don't expect queries, but some are made, the
spec will fail with a list of the queries made. This makes debugging much
simpler, since you can usually narrow down the query to a missing `includes`.

## Introducing the `db-query-matchers` gem

We bundled the `make_database_queries` matcher into a
[reusable gem][db-query-matchers]. Next time you add a caching layer or fix an
N+1 query, make sure to add the performance improvement to your test suite!

Check out the project on Github:
[https://github.com/causes/db-query-matchers][db-query-matchers]

[profile page]: https://www.causes.com/henric
[Joe]: http://joelencioni.com
[ActiveRecord]: http://api.rubyonrails.org/classes/ActiveRecord/Base.html
[db-query-matchers]: https://github.com/causes/db-query-matchers
[RSpec]: http://rspec.info/
[causes.com]: https://www.causes.com/
