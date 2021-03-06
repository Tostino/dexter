# Dexter

The automatic indexer for Postgres

[Read about how it works](https://medium.com/@ankane/introducing-dexter-the-automatic-indexer-for-postgres-5f8fa8b28f27)

[![Build Status](https://travis-ci.org/ankane/dexter.svg?branch=master)](https://travis-ci.org/ankane/dexter)

## Installation

First, install [HypoPG](https://github.com/dalibo/hypopg) on your database server. This doesn’t require a restart.

```sh
cd /tmp
curl -L https://github.com/dalibo/hypopg/archive/1.0.0.tar.gz | tar xz
cd hypopg-1.0.0
make
make install # may need sudo
```

> Note: If you have issues, make sure `postgresql-server-dev-*` is installed.

Enable logging for slow queries in your Postgres config file.

```ini
log_min_duration_statement = 10 # ms
```

And install the command line tool with:

```sh
gem install pgdexter
```

The command line tool is also available as a [Linux package](guides/Linux.md).

## How to Use

Dexter needs a connection to your database and a log file to process.

```sh
tail -F -n +1 <log-file> | dexter <connection-options>
```

This finds slow queries and generates output like:

```
Started
Processing 189 new query fingerprints
Index found: genres_movies (genre_id)
Index found: genres_movies (movie_id)
Index found: movies (title)
Index found: ratings (movie_id)
Index found: ratings (rating)
Index found: ratings (user_id)
Processing 12 new query fingerprints
```

To be safe, Dexter will not create indexes unless you pass the `--create` flag. In this case, you’ll see:

```
Index found: ratings (user_id)
Creating index: CREATE INDEX CONCURRENTLY ON "ratings" ("user_id")
Index created: 15243 ms
```

## Connection Options

Dexter supports the same connection options as psql.

```
-h host -U user -p 5432 -d dbname
```

This includes URIs:

```
postgresql://user:pass@host:5432/dbname
```

and connection strings:

```
host=localhost port=5432 dbname=mydb
```

## Options

Name | Description | Default
--- | --- | ---
exclude | prevent specific tables from being indexed | None
interval | time to wait between processing queries, in seconds | 60
log-level | `debug` gives additional info for suggested indexes<br />`debug2` gives additional info for processed queries<br />`error` suppresses logging | info
log-sql | log SQL statements executed | false
min-time | only process queries consuming a min amount of DB time, in minutes | 0

## Non-Streaming Modes

You can pass a single statement with:

```sh
dexter <connection-options> -s "SELECT * FROM ..."
```

or files with:

```sh
dexter <connection-options> <file1> <file2>
```

## Examples

Ubuntu with PostgreSQL 9.6

```sh
tail -F -n +1 /var/log/postgresql/postgresql-9.6-main.log | sudo -u postgres dexter dbname
```

Homebrew on Mac

```sh
tail -F -n +1 /usr/local/var/postgres/server.log | dexter dbname
```

## Hosted Postgres

Some hosted providers like Amazon RDS and Heroku do not support the HypoPG extension, which Dexter needs to run. See [how to use Dexter](guides/Hosted-Postgres.md) in these cases.

## Future Work

[Here are some ideas](https://github.com/ankane/dexter/issues/1)

## Upgrading

Run:

```sh
gem install pgdexter
```

To use master, run:

```sh
gem install specific_install
gem specific_install ankane/pgdexter
```

## Thanks

This software wouldn’t be possible without [HypoPG](https://github.com/dalibo/hypopg), which allows you to create hypothetical indexes, and [pg_query](https://github.com/lfittl/pg_query), which allows you to parse and fingerprint queries. A big thanks to Dalibo and Lukas Fittl respectively.

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/dexter/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/dexter/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features

To get started, run:

```sh
git clone https://github.com/ankane/dexter.git
cd dexter
bundle
rake install
```

To run tests, use:

```sh
rake test
```
