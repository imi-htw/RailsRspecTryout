
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


See aeac88ec7b9bd42000b9dbdbc3dcba2e12bcbbd5 for the changes I made.

## Testing locally

Change the config/database.yml to run sqlite in memory for the test environment (as described in the [Travis Documentation](https://docs.travis-ci.com/user/database-setup/#SQLite3)):

    #database: db/test.sqlite3
    database: ":memory:"
    timeout: 500

## Versions issue

The application "thinks" that not all migrations have been applied. The difference I found lies within the method insert_versions_sql; the insert is split up into several statements if the db doesn't support multi_insert.

    schema_statements.rb

    def insert_versions_sql(versions) # :nodoc:
      sm_table = ActiveRecord::Migrator.schema_migrations_table_name

      if supports_multi_insert?
        sql = "INSERT INTO #{sm_table} (version) VALUES "
        sql << versions.map {|v| "('#{v}')" }.join(', ')
        sql << ";\n\n"
        sql
      else
        versions.map { |version|
          "INSERT INTO #{sm_table} (version) VALUES ('#{version}');"
        }.join "\n\n"
      end
    end


    and

    def supports_multi_insert?
      sqlite_version >= '3.7.11'
    end



<pre>
(0.1ms)  CREATE TABLE "vwords" ("id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, "entry" varchar, "created_at" datetime NOT NULL, "updated_at" datetime NOT NULL)

(0.1ms)  CREATE TABLE "schema_migrations" ("version" varchar NOT NULL PRIMARY KEY)

(0.1ms)  SELECT version FROM "schema_migrations"

(0.1ms)  INSERT INTO "schema_migrations" (version) VALUES ('20161108150237')

(0.1ms)  INSERT INTO schema_migrations (version) VALUES ('20161025142613');

INSERT INTO schema_migrations (version) VALUES ('20161025142620');

INSERT INTO schema_migrations (version) VALUES ('20161025142649');

INSERT INTO schema_migrations (version) VALUES ('20161031134630');

(0.2ms)  CREATE TABLE "ar_internal_metadata" ("key" varchar NOT NULL PRIMARY KEY, "value" varchar, "created_at" datetime NOT NULL, "updated_at" datetime NOT NULL)
</pre>
