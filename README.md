
[![Build Status](https://travis-ci.org/imi-htw/RailsRspecTryout.svg?branch=master)](https://travis-ci.org/imi-htw/RailsRspecTryout)

# Running RSpec Tests against SQLite in Memory, especially on Travis CI

This is a simple app (one resource Note with title and content) to try
out running [rspec](http://rspec.info/documentation/) on Travis CI.

Travis CI requires SQLite to run in memory. Or, at least advises to setup
the test db in memory: [https://docs.travis-ci.com/user/database-setup/#SQLite3](https://docs.travis-ci.com/user/database-setup/#SQLite3)

RSpec can run with SQLite in memory. The problem is that the schema has to be loaded in the same process, so first calling

   rails db:test:prepare (or similar)

doesn't always work, as the db will be deleted again.

I've seen inconsistent behaviour - e.g. that the build runs fine even though; that a second migration fails complaining that tables are still there (which is really strange as the table seems to be there, but not the version information).

I've found hints on how to configure rspec with in memory sqlite [here](https://gist.github.com/brundage/4091314) and (here)[http://philippe.bourgau.net/simplest-way-to-speed-up-rspec-with-in-memory-sqlite-db/]

(Note that [silence_stream](http://apidock.com/rails/Kernel/silence_stream) [has been removed from rails](https://github.com/pat/combustion/pull/64))

## Testing locally

Change the config/database.yml to run sqlite in memory for the test environment (as described in the [Travis Documentation](https://docs.travis-ci.com/user/database-setup/#SQLite3)):

#database: db/test.sqlite3
database: ":memory:"
timeout: 500
