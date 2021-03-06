# Optiq HOWTO

Here's some miscellaneous documentation about using Optiq and its various
adapters.

## Building from a source distribution

Prerequisites are maven (3.0.4 or later)
and Java (JDK 1.6 or later, 1.8 preferred) on your path.

Unpack the source distribution `.tar.gz` or `.zip` file,
`cd` to the root directory of the unpacked source,
then build using maven:

```bash
$ tar xvfz apache-optiq-0.9.0-incubating-source.tar.gz
$ cd apache-optiq-0.9.0-incubating
$ mvn install
```

[Running tests](HOWTO.md#running-tests) describes how to run more or fewer
tests.

## Building from git

Prerequisites are git, maven (3.0.4 or later)
and Java (JDK 1.6 or later, 1.8 preferred) on your path.

Create a local copy of the github repository,
`cd` to its root directory,
then build using maven:

```bash
$ git clone git://github.com/apache/incubator-optiq.git
$ cd incubator-optiq
$ mvn install
```

[Running tests](HOWTO.md#running-tests) describes how to run more or fewer
tests.

## Running tests

The test suite will run by default when you build, unless you specify
`-DskipTests`:

```bash
$ mvn -DskipTests clean install
```

There are other options that control which tests are run, and in what
environment, as follows.

* `-Doptiq.test.db=DB` (where db is `hsqldb` or `mysql`) allows you
  to change the JDBC data source for the test suite. Optiq's test
  suite requires a JDBC data source populated with the foodmart data
  set.
   * `hsqldb`, the default, uses an in-memory hsqldb database.
   * `mysql` uses a MySQL database in `jdbc:mysql://localhost/foodmart`.
     It is somewhat faster than hsqldb, but you need to populate it
     manually.
* `-Doptiq.debug` prints extra debugging information to stdout.
* `-Doptiq.test.slow` enables tests that take longer to execute. For
  example, there are tests that create virtual TPC-H and TPC-DS schemas
  in-memory and run tests from those benchmarks.
* `-Doptiq.test.mongodb=true` enables tests that run against
  MongoDB. MongoDB must be installed, running, and
  [populated with the zips.json data set](HOWTO.md#mongodb-adapter).
* `-Doptiq.test.splunk=true` enables tests that run against Splunk.
  Splunk must be installed and running.

## Contributing

We welcome contributions.

If you are planning to make a large contribution, talk to us first! It
helps to agree on the general approach. Log a
[JIRA case](https://issues.apache.org/jira/browse/OPTIQ) for your
proposed feature or start a discussion on the dev list.

Fork the github repository, and create a branch for your feature.

Develop your feature and test cases, and make sure that `mvn clean
install` succeeds. (Run extra tests if your change warrants it.)

Commit your change to your branch, and use a comment that starts with
the JIRA case number, like this:

```
[OPTIQ-345] AssertionError in RexToLixTranslator comparing to date literal
```

If your change had multiple commits, use `git rebase -i master` to
combine them into a single commit, and to bring your code up to date
with the latest on the main line.

Then push your commit(s) to github, and create a pull request from
your branch to the incubator-optiq master branch. Update the JIRA case
to reference your pull request, and a committer will review your
changes.

## Tracing

To enable tracing, add the following flags to the java command line:

```
-Doptiq.debug=true -Djava.util.logging.config.file=core/src/test/resources/logging.properties
```

The first flag causes Optiq to print the Java code it generates
(to execute queries) to stdout. It is especially useful if you are debugging
mysterious problems like this:

```
Exception in thread "main" java.lang.ClassCastException: Integer cannot be cast to Long
  at Baz$1$1.current(Unknown Source)
```

The second flag specifies a config file for
the <a href="http://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html">java.util.logging</a>
framework. Put the following into core/src/test/resources/logging.properties:

```properties
handlers= java.util.logging.ConsoleHandler
.level= INFO
org.eigenbase.relopt.RelOptPlanner.level=FINER
java.util.logging.ConsoleHandler.level=ALL
```

The line org.eigenbase.relopt.RelOptPlanner.level=FINER tells the planner to produce
fairly verbose outout. You can modify the file to enable other loggers, or to change levels.
For instance, if you change FINER to FINEST the planner will give you an account of the
planning process so detailed that it might fill up your hard drive.

## CSV adapter

See <a href="https://github.com/julianhyde/optiq-csv/blob/master/TUTORIAL.md">optiq-csv
tutorial</a>.

## MongoDB adapter

First, download and install Optiq,
and <a href="http://www.mongodb.org/downloads">install MongoDB</a>.

Import MongoDB's zipcode data set into MongoDB:

```bash
$ curl -o /tmp/zips.json http://media.mongodb.org/zips.json
$ mongoimport --db test --collection zips --file /tmp/zips.json
Tue Jun  4 16:24:14.190 check 9 29470
Tue Jun  4 16:24:14.469 imported 29470 objects
```

Log into MongoDB to check it's there:

```bash
$ mongo
MongoDB shell version: 2.4.3
connecting to: test
> db.zips.find().limit(3)
{ "city" : "ACMAR", "loc" : [ -86.51557, 33.584132 ], "pop" : 6055, "state" : "AL", "_id" : "35004" }
{ "city" : "ADAMSVILLE", "loc" : [ -86.959727, 33.588437 ], "pop" : 10616, "state" : "AL", "_id" : "35005" }
{ "city" : "ADGER", "loc" : [ -87.167455, 33.434277 ], "pop" : 3205, "state" : "AL", "_id" : "35006" }
> exit
bye
```

Connect using the <a href="https://github.com/julianhyde/optiq/blob/master/mongodb/src/test/resources/mongo-zips-model.json">mongo-zips-model.json</a> Optiq model:
```bash
$ ./sqlline
sqlline> !connect jdbc:optiq:model=mongodb/target/test-classes/mongo-zips-model.json admin admin
Connecting to jdbc:optiq:model=mongodb/target/test-classes/mongo-zips-model.json
Connected to: Optiq (version 0.4.x)
Driver: Optiq JDBC Driver (version 0.4.x)
Autocommit status: true
Transaction isolation: TRANSACTION_REPEATABLE_READ
sqlline> !tables
+------------+--------------+-----------------+---------------+
| TABLE_CAT  | TABLE_SCHEM  |   TABLE_NAME    |  TABLE_TYPE   |
+------------+--------------+-----------------+---------------+
| null       | mongo_raw    | zips            | TABLE         |
| null       | mongo_raw    | system.indexes  | TABLE         |
| null       | mongo        | ZIPS            | VIEW          |
| null       | metadata     | COLUMNS         | SYSTEM_TABLE  |
| null       | metadata     | TABLES          | SYSTEM_TABLE  |
+------------+--------------+-----------------+---------------+
sqlline> select count(*) from zips;
+---------+
| EXPR$0  |
+---------+
| 29467   |
+---------+
1 row selected (0.746 seconds)
sqlline> !quit
Closing: net.hydromatic.optiq.jdbc.FactoryJdbc41$OptiqConnectionJdbc41
$
```

## Splunk adapter

To run the test suite and sample queries against Splunk,
load Splunk's `tutorialdata.zip` data set as described in
<a href="http://docs.splunk.com/Documentation/Splunk/6.0.2/PivotTutorial/GetthetutorialdataintoSplunk">the Splunk tutorial</a>.

(This step is optional, but it provides some interesting data for the sample
queries. It is also necessary if you intend to run the test suite, using
`-Doptiq.test.splunk=true`.)

## Implementing an adapter

New adapters can be created by implementing `OptiqPrepare.Context`:

```java
import net.hydromatic.optiq.Schema;
import net.hydromatic.optiq.impl.java.JavaTypeFactory;
import net.hydromatic.optiq.jdbc.OptiqPrepare;
public class AdapterContext implements OptiqPrepare.Context {

    @Override
    public JavaTypeFactory getTypeFactory() {
        // adapter implementation
        return typeFactory;
    }

    @Override
    public Schema getRootSchema() {
        // adapter implementation
        return rootSchema;
    }

}
```

### Testing adapter in Java

The example below shows how SQL query can be submitted to
`OptiqPrepare` with a custom context (`AdapterContext` in this
case). Optiq prepares and implements the query execution, using the
resources provided by the `Context`. `OptiqPrepare.PrepareResult`
provides access to the underlying enumerable and methods for
enumeration. The enumerable itself can naturally be some adapter
specific implementation.

```java
import net.hydromatic.optiq.jdbc.OptiqPrepare;
import net.hydromatic.optiq.prepare.OptiqPrepareImpl;
import org.junit.Test;

public class AdapterContextTest {

    @Test
    public void testSelectAllFromTable() {
        AdapterContext ctx = new AdapterContext();
        String sql = "SELECT * FROM TABLENAME";
        Type elementType = Object[].class;
        OptiqPrepare.PrepareResult<Object> prepared = new OptiqPrepareImpl()
                .prepareSql(ctx, sql, null, elementType, -1);
        Object enumerable = prepared.getExecutable();
        // etc.
    }

}
```

## JavaTypeFactory

When Optiq compares `Type` instances, it requires them to be the same
object. If there are two distinct `Type` instances that refer to the
same Java type, Optiq may fail to recognize that they match.  It is
recommended to:
-   Use a single instance of `JavaTypeFactory` within the optiq context
-   Store the `Type` instances so that the same object is always returned for the same `Type`.

## Set up PGP signing keys (for Optiq committers)

Follow instructions at http://www.apache.org/dev/release-signing to
create a key pair. (On Mac OS X, I did `brew install gpg` and `gpg
--gen-key`.)

Add your public key to the `KEYS` file by following instructions in
the `KEYS` file.

## Making a snapshot (for Optiq committers)

Before you start:
* Set up signing keys as described above.
* Make sure you are using JDK 1.7 (not 1.6 or 1.8).
* Make sure build and tests succeed with `-Doptiq.test.db=hsqldb` (the default)

```bash
# set passphrase variable without putting it into shell history
read GPG_PASSPHRASE

# make sure that there are no junk files in the sandbox
git clean -x

mvn clean install -Prelease,apache-release -Dgpg.passphrase=${GPG_PASSPHRASE}
```

When the dry-run has succeeded, change `install` to `deploy`.

## Making a release (for Optiq committers)

Before you start:
* Set up signing keys as described above.
* Make sure you are using JDK 1.7 (not 1.6 or 1.8).
* Make sure build and tests succeed, including with
  -Doptiq.test.db={mysql,hsqldb}, -Doptiq.test.slow=true,
  -Doptiq.test.mongodb=true, -Doptiq.test.splunk=true.

```bash
# set passphrase variable without putting it into shell history
read GPG_PASSPHRASE

# make sure that there are no junk files in the sandbox
git clean -x

```

Check the artifacts:
* Make sure that binary and source distros have a README file
  (README.md does not count) and that the version in the README is
  correct
* The file name must start `apache-optiq-` and include `incubating`.
* Check PGP, per https://httpd.apache.org/dev/verification.html

