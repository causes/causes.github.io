---
layout: post
title:  "Even Faster: Loading Half a Billion Rows in MySQL Revisited"
date:   2012-08-21
author: Adam Derewecki
categories:
  - MySQL
---

A few months ago, I wrote a post on loading 500 million rows into a single
innoDB table from flatfiles. This was in the effort to un-‘optimize’ a premature
optimization in our codebase: user action credits were being stored in monthly
sharded tables to keep the tables small and performant. As our use of the code
changed, we found more and more that we had to do a query for each month to see
if a user had taken an action. We implemented some performance optimizations
(mainly memcaching values from prior months as they are immutable), but it was
still overly complicated and prone to bugs. Since we had another table that was
900m rows, it seemed reasonable to collapse these shards into one 500m row
table.

Since writing the last post, I’ve learned that there’s a much quicker way to
combine those tables — as long as you already have the data in MySQL. MySQL
allows for selecting from one table into another via the INSERT INTO ..
SELECT statement:

```sql
INSERT INTO dest_table (values) SELECT values FROM source_table;
```

which might look something like:

```sql
INSERT INTO credits_new (user_id, activity_id, created_at)
SELECT user_id, activity_id, created_at FROM credits;
```

This shouldn’t be surprising; an ALTER TABLE on an innoDB table creates a new
table with the new schema and copies the rows from the old table over to the
new table.

With large tables though, you still have to do a little leg work to get this to
work properly. First, the database machine needs enough space to hold both the
old table(s) and new tables on disk, until you’re able to delete the old table.
As you insert into the table, inserts will get slower and slower. This also
makes sense; as the table grows the indexes grow, and working with them gets
slower. You’ll want to break the loading into many chunks so that each
transaction completes in a reasonable amount of time. In practice, I’ve found
that the optimum chunk size gets smaller as the size of the table grows.
Starting at 100k rows is not unreasonable, but don’t be surprised if near the
end you need to drop it closer to 10k or 5k to keep the transaction to under 30
seconds. If you don’t keep the transaction time reasonable, the whole operation
could outright fail eventually with something like:

```sql
ERROR 1197 (HY000): Multi-statement transaction required more than
‘max_binlog_cache_size’ bytes of storage; increase this mysqld variable and
try again.
```

It will certainly slow down (I wish I had kept a graph of timestamp vs rows
inserted, but it’s certainly de-motivating). You might also want a delay as to
not overwhelm the slave and cause replication to lag too far behind. I used
external files to store the values of both variables that were read from disk at
each iteration, so that I could change the values without interrupting the
script. Using the same file trick, each iteration wrote out the table name and
id of the last row read (in case something broke and I wanted to resume).

This is the script that was used to perform the migration. I left it running in
a tmux session and also used `watch -n 10 LAST_WRITTEN’ to monitor the progress.

```ruby
#!/usr/bin/env ruby

def file_variable(filename)
  File.readlines(filename, 'r').first.to_i
end

TARGET_TABLE = 'action_credits'
SCHEMA = <<-SQL
CREATE TABLE IF NOT EXISTS `action_credits` (
.. omitted for brevity ..
 ) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
SQL
insert_columns = (has_utm_campaign ? columns : columns + ['utm_campaign']).join ','
select_columns = (has_utm_campaign ? columns : columns + ['NULL']).join ','

chunk_size = file_variable('CHUNK_SIZE')
(0..(rows / chunk_size)).each do |x|
  lower = x * chunk_size
  upper = lower + chunk_size - 1
  query = <<-SQL.squish
    INSERT INTO \#{TARGET_TABLE} (\#{insert_columns})
    SELECT \#{select_columns} FROM \#{table}
    WHERE id BETWEEN \#{lower} AND \#{upper}
  SQL
  query_start = Time.now
  q(query, true)
  puts "Finished at \#{Time.now} in \#{Time.now - query_start} seconds"

  File.open('LAST_WRITTEN', 'w+') {|f| f.write([table, upper].join(','))}

  delay = file_variable('DELAY')
  puts "Waiting \#{delay} seconds" if delay > 0
  sleep delay
end
end
puts "Finished in \#{((Time.now - start) / 3600).to_i} hours"
```

There’s a slight concern that dropping the table while the filesystem deletes
the ibd files (where the innoDB data lives) will lock the table for a long
period of time (see http://bugs.mysql.com/bug.php?id=41158), but it wasn’t a
problem when I tried it on a 116 GB table. If you prefer to be paranoid (like I
was), there’s a trick you can use to unlink the table data asynchronously:
create a hard link to the table’s .ibd file before you DROP the table; the DROP
will only unlink one of the two hard links to the file. Afterward you can `rm
the_hardlink’ and your filesystem will remove the .ibd data. Using this method
in practice took me 5 seconds to DROP and 13 seconds to rm the hardlink.

If all you need to do is a simple alter table (and not something complicated
like combining sharded tables), I’d recommend using the pt-online-schema-change
tool provided by Percona. We’ll look at the details of that tool in a future
post.
