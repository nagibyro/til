# Stream DB Changes
One thing that I've really enjoyed with DynamoDB in AWS has been the ability to stream database
changes and do things on create, update or delete of records. This never seemed to be overly easy to do with
a traditional RDMS. However with Postgres there is a DB extension called [wal2json](https://github.com/eulerto/wal2json)
which translates Postgres's [WAL Log](https://www.postgresql.org/docs/current/wal-intro.html) to json format that makes
it easy for apps to consume.

The basic premise is you can write an application that attaches to the main DB essentially like a read replica and will
read the [WAL Log](https://www.postgresql.org/docs/current/wal-intro.html) which holds DB actions such as insert,
update, delete, and truncate. From their you can write logic to handle that record change like push to Elasticsearch or
put a message on a queue or whatever you can think of. While this may not be as simple as DynamoDB Streams it shows
that building a system to stream out of Postgres is not out of the realm of possibility. Though may require gaining a
bit more knowledge of how Postgres works.

## Note
AWS RDS supports the `wal2json` extension in it's manged Postgres offering. Note however this doesn't work
with Aurora Postgres since thats just a wire compatible Postgres but proprietary storage engine.