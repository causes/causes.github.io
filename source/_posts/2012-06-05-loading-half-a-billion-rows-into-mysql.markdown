---
layout: post
title:  Loading half a billion rows into MySQL
author: Adam Derewecki
date:   2012-06-05
categories:
  - MySQL
---

## Background

We have a legacy system in our production environment that keeps track of when
a user takes an action on Causes.com (joins a Cause, recruits a friend, etc).
I say legacy, but I really mean a prematurely-optimized system that I'd like
to make less smart. This 500m record database is split across monthly sharded
tables. Seems like a great solution to scaling (and it is)&mdash;except that
we don't need it. And based on our usage pattern (e.g. to count a user's total
number of actions, we need to do query N tables), this leads to pretty severe
performance degradation issues. Even with memcache layer sitting in front of
old month tables, new features keep discovering new N-query performance
problems.  Noticing that we have another database happily chugging along with
900 million records, I decided to migrate the existing system into a single
table setup. The goals were:

  - Reduce complexity. Querying one table is simpler than N tables.
  - Push as much complexity as possible to the database. The wrappers around the
    month-sharding logic in Rails are slow and buggy.
  - Increase performance. Also related to one table query being simpler than N.


## Alternative Proposed Solutions

*MySQL Partitioning:*
This was the most similar to our existing set up, since MySQL internally
stores the data into different tables. We decided against it because it seemed
likely that it wouldn't be much faster than our current solution (although
MySQL can internally do some optimizations to make sure you only look at
tables that could possibly have data you want). And it's still the same
complexity we were looking to reduce (and would further be the only database
set up in our system using partitioning).

*Redis:*
Not really proposed as an alternative because the full dataset won't fit
into memory, but something we're considering loading a subset of the data into
to answer queries that we make a lot that MySQL isn't particularly good at
(e.g.  'which of my friends have taken an action' is quick using Redis's built
in `SET UNION` function). The new MySQL table might be performant enough that it
doesn't make sense to build a fast Redis version, so we're avoiding this as
possible premature optimization, especially with a technology we're not as
familiar with.

<!-- more -->

## Dumping the old data

MySQL provides the `mysqldump` utility to allow quick dumping to disk:

```bash
mysqldump -T /var/lib/mysql/database_data database_name
```

This will produce a TSV file for each table in the database, and this is the
format that `LOAD INFILE` will be able to quickly load later on.

## Installing Percona 5.5

We'll be building the new system with the latest-and-greatest in Percona
databases on CentOS 6.2:

```bash
rpm -Uhv http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm
yum install Percona-Server-shared-compat Percona-Server-client-55 Percona-Server-server-55 -y
```

[ open bug with the compat package:
<https://bugs.launchpad.net/percona-server/+bug/908620> ]

## Specify a directory for the InnoDB data

This isn't exactly a performance tip, but I had to do some digging to get MySQL
to store data on a different partition. The first step is to make use your
my.cnf contains a

    datadir = /path/to/data

directive. Make sure /path/to/data is owned by mysql:mysql

```bash
chown -R mysql.mysql /path/to/data
```
and run:

```bash
mysql_install_db --user=mysql --datadir=/path/to/data
```

This will set up the directory structures that InnoDB uses to store data. This
is also useful if you're aborting a failed data load and want to wipe the slate
clean (if you don't specify a directory, /var/lib/mysql is used by default).
Just `rm -rf *` the directory and run the `mysql_install_db` command.

[* <http://dev.mysql.com/doc/refman/5.5/en/mysql-install-db.html> ]


## SQL Commands to Speed up the LOAD DATA

You can tell MySQL to not enforce foreign key and uniqueness constraints:

```sql
SET FOREIGN_KEY_CHECKS = 0;
SET UNIQUE_CHECKS = 0;
```

and drop the transaction isolation guarantee to UNCOMMITTED:

```sql
SET SESSION tx_isolation='READ-UNCOMMITTED';
```

and turn off the binlog with:

```sql
SET sql_log_bin = 0;
```

And when you're done, don't forget to turn it back on with:

```sql
SET UNIQUE_CHECKS = 1;
SET FOREIGN_KEY_CHECKS = 1;
SET SESSION tx_isolation='READ-REPEATABLE';
```

It's worth noting that a lot of resources will tell you to to use the "DISABLE
KEYS" directive and have the indices all built once all the data has been loaded
into the table. Unfortunately, InnoDB does not support this. I tried it, and
while it took only a few hours to load 500m rows, the data was unusable without
any indices. You could drop the indices completely and add them later, but with
a table size this big I didn't think it would help much.

Another red herring was turning off autocommit and committing after each `LOAD
DATA` statement. This was effectively the same thing as autocommitting, and
manually commiting led to `LOAD DATA` slowdowns a quarter of the way in.
[http://dev.mysql.com/doc/refman/5.1/en/alter-table.html]

    - [<http://dev.mysql.com/doc/refman/5.1/en/alter-table.html>, search for
      'DISABLE KEYS']
    - [ <http://www.mysqlperformanceblog.com/2007/11/01/innodb-performance-optimization-basics/> ]

## Performance adjustments made to my.cnf

```sql
-flush_log='http://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit'
-- #{ link_to flush_log, flush_log }
-- this loosens the frequency with which the data is flushed to disk
-- it's possible to lose a second or two of data this way in the event of a
-- system crash, but this is in a very controlled circumstance
innodb_flush_log_at_trx_commit=2
-- rule of thumb is 75% - 80% of total system memory
innodb_buffer_pool_size=16GB
-- don't let the OS cache what InnoDB is caching anyway
-- http://www.mysqlperformanceblog.com/2007/11/01/innodb-performance-optimization-basics/
innodb_flush_method=O_DIRECT
-- don't double write the data
-- http://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_doublewrite
innodb_doublewrite = 0
```

## Use LOAD DATA INFILE

This is the most optimized path toward bulk loading structured data into MySQL.
[8.2.2.1. Speed of `INSERT` Statements,](http://dev.mysql.com/doc/refman/5.5/en/insert-speed.html)
predicts a ~20x speedup over a bulk `INSERT` (i.e. an `INSERT` with thousands of
rows in a single statement). See also
[8.5.4. Bulk Data Loading for InnoDB Tables,](http://dev.mysql.com/doc/refman/5.5/en/optimizing-innodb-bulk-data-loading.html)
for a few more tips.

Not only is it faster, but in my experience with this migration, the INSERT
method will slow down faster than it can load data and effectively never finish
(last estimate I made was 60 days, but it was still slowing down).

INFILE must be in the directory that InnoDB is storing that database
information. If MySQL is in /var/lib/mysql, then mydatabase would be in
/var/lib/mysql/mydatabase. If you don't have access to that directory on the
server, you can use `LOAD DATA LOCAL INFILE`. In my testing, putting the file in
the proper place and using `LOAD DATA INFILE` increased load performance by
about 20%.

<http://dev.mysql.com/doc/refman/5.5/en/load-data.html>


## Perform your data transformation directly in MySQL

Our old actioncredit system was unique on (MONTH(created_at), id), but the new
system is going to generate new autoincrementing IDs for each records as it's
loaded in chronological order. The problem was that my 50 GB of TSV data doesn't
match up to the new schema. Some scripts I had that would use Ruby to transform
the old row into the new row was laughably slow. I did some digging and found
out that you can tell MySQL to (quickly) throw away the data you don't want in
the load statement itself, using parameter binding:

```sql
LOAD DATA INFILE 'data.csv' INTO TABLE mytable
FIELDS TERMINATED by '\t' ENCLOSED BY '\"'
(@throwaway), user_id, action, created_at
```

This statement is telling MySQL which fields are represented in data.csv.
@throwaway is a binding parameter; and in this case we want to discard it so
we're not going to bind it. If we wanted to insert a prefix, we could execute:

```sql
LOAD DATA INFILE 'data.csv' INTO TABLE mytable
FIELDS TERMINATED by '\t' ENCLOSED BY '\"'
(id, user_id, @action, created_at
SET action=CONCAT('prefix_', action)
```

and every loaded row's `action' column will begin with the string 'prefix'.


## Checking progress without disrupting the import

If you're loading large data files and want to check the progress, you
definitely don't want to use `SELECT COUNT(*) FROM table'. This query will
degrade as the size of the table grows and slowdown the LOAD process. Instead
you can query:

```sql
> SELECT table_rows FROM information_schema.tables WHERE table_name = 'table';
+------------+
| table_rows |
+------------+
|   27273886 |
+------------+
1 row in set (0.23 sec)
```

If you want to watch/log the progress over time, you can craft a quick shell
command to poll the number of rows:

```bash
$ while :; do mysql -hlocalhost databasename -e "SELECT table_rows FROM information_schema.tables WHERE table_name = 'table' \G ; " | grep rows | cut -d':' -f2 | xargs echo `date +"%F %R"` , | tee load.log && sleep 30; done
2012-05-29 18:16 , 32267244
2012-05-29 18:16 , 32328002
2012-05-29 18:17 , 32404189
2012-05-29 18:17 , 32473936
2012-05-29 18:18 , 32543698
2012-05-29 18:18 , 32616939
2012-05-29 18:19 , 32693198
```

The `tee` will echo to STDOUT as well as to `file.log`, the `\G` formats the
columns in the result set as rows, and the sleep gives it a pause between
loading.

## LOAD DATA chunking script

I quickly discovered that throwing a 50m row TSV file at LOAD DATA was a good
way to have performance degrade to the point of not finishing. I settled on
using `split' to chunk data into one million rows per file:

```bash
for month_table in action*.txt; do
echo "$(date) splitting $month_table..."
split -l 1000000 $month_table curmonth_
for segment in curmonth_*; do
  echo "On segment $segment"
  time mysql -hlocalhost action_credit_silo <<-SQL
    SET FOREIGN_KEY_CHECKS = 0;
    SET UNIQUE_CHECKS = 0;
    SET SESSION tx_isolation='READ-UNCOMMITTED';
    SET sql_log_bin = 0;
    LOAD DATA INFILE '$segment' INTO TABLE actioncredits
    FIELDS TERMINATED by '\t' ENCLOSED BY '\"'
    (@throwawayId, action, user_id, target_user_id, cause_id, item_type, item_id, activity_id, created_at, utm_campaign) ;
  SQL
    rm $segment
  done
  mv $month_table $month_table.done
done
```


## Wrap-up

Over the duration of this script, I saw chunk load time increase from 1m40s to
around an hour per million inserts. This is however better than not finishing at
all, which I wasn't able to achieve until making all changes suggested in this
post and using the aforementioned `load.sh' script. Other tips:

  - use as few indices as you can
  - loading the data in sequential order not only makes the loading faster, but
    the resulting table will be faster
  - if you can load any of the data from MySQL (instead of a flat file
    intermediary), it will be much faster. You can use the `INSERT INTO ..
    SELECT' statement to copy data between tables quickly.

UPDATE: Since writing this article, I've found even faster ways to load
this kind of data.  You can read more in the follow-up:

[Even faster: loading half a billion rows in MySQL revisited'](http://derwiki.tumblr.com/post/29892583773/even-faster-loading-half-a-billion-rows-in-mysql)

Thanks for reading drafts of this to Greg and Lann, two of my
super-smart coworkers at Causes. Check out
[causes.com/jobs](http://www.causes.com/jobs) if this sort of work interests you!
