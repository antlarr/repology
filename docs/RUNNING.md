# Running repology

## Dependencies

Needed for core:

- [Python](https://www.python.org/) 3.6+
- Python module [pyyaml](http://pyyaml.org/)
- Python module [requests](http://python-requests.org/)
- Python module [rubymarshal](https://github.com/d9pouces/RubyMarshal)
- [libversion](https://github.com/repology/libversion) library

Needed for fetching/parsing specific repository data:

- [wget](https://www.gnu.org/software/wget/)
- [git](https://git-scm.com/)
- [rsync](https://rsync.samba.org/)
- [librpm](http://www.rpm.org/) and [pkg-config](https://www.freedesktop.org/wiki/Software/pkg-config/)

Needed for web-application:

- Python module [flask](http://flask.pocoo.org/)
- Python module [psycopg](http://initd.org/psycopg/)
- [PostgreSQL database](https://www.postgresql.org/) 9.6+

Optional, for doing HTML validation when running tests:
- Python module [pytidylib](https://pypi.python.org/pypi/pytidylib) and [tidy-html5](http://www.html-tidy.org/) library

## Building

Though repology is mostly a Python project, it contains C utility to
read binary rpm format, which is used for parsing ALT Sisyphus
repository. To build the utility, run ```make``` in project root.
You need librpm and pkg-config.

## Usage

### Configuration

First, you may need to tune settings which are shared by all repology
utilities, such as directory for storing downloaded repository state
or DSN (string which specifies how to connect to PostgreSQL database).
See ```repology.conf.default``` for default values, create
```repology.conf``` in the same directory to override them or
specify path to alternative config in ```REPOLOGY_SETTINGS```
environment variable, or override settings via command line.

By default, repology uses ```./_state``` state directory and
```repology/repology/repology``` database/user/password on localhost.

### Fetching/updating repository data

First, let's try to fetch some repository data and see if it works.
No database is needed at this point.

```
./repology-update --fetch --parse
```

* ```--fetch``` tells the utility to fetch raw repository data
(download files, scrape websites, clone git repos) into state
directory. Note that it won't refetch (update) data unless
```--update``` is also specified.
* ```--parse``` parses downloaded raw data into internal format
which is also saved into state directory.

After data is downloaded you may inspect it with

```
./repology-dump.py | less
```

The utility allows filtering and several modes of operation, see
```--help``` for full list of options.

### Creating the database

To run repology webapp you need PostgreSQL database.

First, ensure PostgreSQL server is installed and running,
and execute the following SQL queries (usually you'll run
```psql -U postgres``` for this):

```
CREATE DATABASE repology;
CREATE USER repology WITH PASSWORD 'repology';
GRANT ALL ON DATABASE repology TO repology;
```

now you can create database schema (tables, indexes etc.) with:

```
./repology-update --initdb
```

and finally push parsed data into the database with:

```
./repology-update --database
```

### Running the webapp

Repology is a flask application, so as long as you've set up
database and configuration, you may just run the application
locally:

```
./repology-app.py
```

and point your browser to http://127.0.0.1:5000/ to view the
site. This should be enough for personal usage, experiments and
testing.

Alternatively, you may deploy the application in numerous ways,
including mod_wsgi, uwsgi, fastcgi and plain CGI application. See
[flask documentation on deployment](http://flask.pocoo.org/docs/0.11/deploying/)
for more info.

### Keeping up to date

To keep repository data up to date, you'll need to periodically
refetch it and update the database. Note there's no need to recreate
database schema every time (unless it needs changing). Everything
can be done with single command:

```
./repology-app.py --fetch --update --parse --database
```
