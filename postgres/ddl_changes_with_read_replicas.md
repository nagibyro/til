# DDL Changes and Read Replicas 
In a previous TIL I talked about [migration locks](./migration_locks.md) and
how running DDL statements like `ALTER TABLE` can aquire locks. When my
coworkers and I were dicussing migration locks and how to make our DB
migrations in CI safer, I was asked what effect does DDL Locks have on read
replicas. I didn't know the answer at the time but was keen to find out. 

Turns out (as usual) that the excellent postgres documentation has an entire
section talking about [query conflicts](https://www.postgresql.org/docs/current/hot-standby.html#HOT-STANDBY-CONFLICT)
in read replicas (or Hot Standby's as they are called in postgres documentation).
Read replica's work by having the WAL log shipped from the primary to the
replica. Since the work that has happened in the WAL has already happened on
the primary replica's MUST apply the changes. In this case replica's have
configuration options `max_standby_archive_delay` and
`max_standby_streaming_delay` that define how long a read replica will wait for
queries to complete before apply the WAL. (In our RDS instance this was
defaulted to 14 seconds).

Once that WAL delay time has expired the read replica will cancel running queries
and depending on the type of change may even disconnect the postgres connection
from the client. Then apply the WAL change.

The take away for us is that even code like background and analytics jobs that
were only using read replicas needed to be prepared to retry queries that might
get cancelled during a DB migration.
