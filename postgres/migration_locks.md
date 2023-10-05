# Migration Locks
DB migrations on a heavily used DB is one of the leading causes of outages for
my team in th last year. This is usually from a migration acquiring a lock. 
Migration almost always acquire some form of lock on the table they are
modifying. This is important to understand because I didn't have a strong
mental model for how these locks can interact with other concurrent
transactions.

Most of the time when we think about migrations locking we think about
migrations having a lock on the table and doing some work. If that work takes
too long to complete you might cause issues for your app or background processes 
that support your app. However acquiring a lock for a migration can impact your
site even if the migration does almost nothing. Like adding a new column with a
null default.

The reason for this is because while the migration tries to acquire it's lock.
It also stops other transactions (other background process or web requests)
from working on table its trying to modify. But if you have other processes
with long running transactions the migration will not be able to run but it'll
prevent other things from running while it waits. So in my experince migrations
cause outages for two main reasons:

1. The migration runs acquires a lock and it has a lot of work to do, rewriting
   an index or writing data to a new column etc...
2. The migration can't acquire it's lock because of some other long running
   transaction and all other work gets stuck behind the migration waiting
   for it while the migration waits on something slow.

The second reason is really just a side effect of the evil of long running
transactions. In general you want to keep transactions short and sweet.

## Mitigations
It's generally a good idea that in your migration transaction to `SET lock_timeout to '3s';`
this makes sure your migration doesn't wait forever to get its lock and will
fail instead of holding up all the work indefinitely. It's also a good idea to
`SET statement_timeout to '6s';` to make sure that if the transaction turns out
to be slow (because of reason 1) that your outage will only be a small blip
instead of multiple minutes or longer. These two settings are simple and will save you a lot
of pain with in your migrations.

## Bonus See what is locked
“Knowing where the trap is—that's the first step in evading it.” To check what's
locking tables and if the transaction has been running for a long time you
can run:

```sql
SELECT
psa.pid as pid,
  psa.datname as database,
  psa.query as current_query,
  clock_timestamp() - psa.xact_start AS transaction_age,
  array_agg(distinct c.relname) AS tables_with_locks
FROM pg_catalog.pg_stat_activity psa
JOIN pg_catalog.pg_locks l ON (psa.pid = l.pid)
JOIN pg_catalog.pg_class c ON (l.relation = c.oid)
JOIN pg_catalog.pg_namespace ns ON (c.relnamespace = ns.oid)
WHERE psa.pid != pg_backend_pid()
  AND ns.nspname != 'pg_catalog'
  AND c.relkind = 'r'
  AND psa.xact_start < clock_timestamp() - '5 seconds'::interval
GROUP BY psa.datname, psa.query, psa.xact_start, psa.pid;
```

If the table you want to migrate is in the results of this query then you need
to either wait for the process to finish or pause the process that's running the
query to complete your migration


## References

- [Postgresql at Scale: Database Schema changes without Downtime](https://medium.com/paypal-tech/postgresql-at-scale-database-schema-changes-without-downtime-20d3749ed680)
- [Pgroll](https://xata.io/blog/pgroll-schema-migrations-postgres)
