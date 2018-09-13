---
layout:      blogpost
title:       Externally Browsing H2 In-Memory Database During A Transaction
img:         001-h2-logo.png
date:        2017-03-01 00:00:00 +0000
tags:        ["H2", "Database", "Debug", "Test"]
excerpt:     A convenient way to fix a defective complex sql-query in a service that causes a functional test to fail is to inspect the database state using a favorite tool while the test is running. Presuming a functional-test uses an in-memory database, there are challenges such as inaccessibility of in-memory database from external applications, isolation of transactions, etc. which must be addressed..
---

## The Situation
* We have a data-intensive system with services that use **complex sql queries**.
* An appropriate automatic test-suite is present for the system and it is mostly consisted of **functional-tests**. Each test sets up an **in-memory database** in the beginning, stores some sample data based on a scenario to the database, invokes the target service, runs different assertions on the result, and finally destroys in-memory database.
* We have faced a **test that fails** (could be either a newly written test, or part of the regression test-suite); The failure rises from one query that does not return the data as expected. We know the faulty query but as it is a complicated query with many joins/aggregations, we have **trouble finding out how the query does not work as expected**.

## Wish
A convenient way to analyze and fix the faulty sql query is to **browse the database with a favorite tool** just when the test is running, the service has been invoked and it has reached the point of the defective query.
At this point the state of database can be carefully inspected, the query be modified and tested online on the database until the correct query that gives the right result is found.

## Limitations
* We would like to stick to in-memory database because it is fast, straghtforward, loads along with the test and destroys itself after the test. However, in-memory databases are often only accessible within the application self and **could not be accessed by any external applications (e.g. a db-browser)**.
* The service might be transactional and consisted of several important steps. We need to carry out database inspection in the middle of the transaction, however the **isolation property of transactions** obscures the changes to other database sessions / transactions up until changes are committed atomically to the database.

![](/img/posts/001-h2-logo.png)

## Solution
H2 covers all the features to grant us our wish:<br/>
- It can work as an in-memory database with a very satisfying performance result.<br/>
{% highlight yml %}
jdbc-url=jdbc:h2:mem:mydatabase;
{% endhighlight %}
- H2 can be programmed to listen to a port, and it will expose the in-memory database to external applications for remotely accessing and browsing the database.<br/>
{% highlight java %}
Server.createTcpServer ("-tcp", "-tcpAllowOthers", "-tcpPort", "9092", "-tcpDaemon");
tcpServer.start();
{% endhighlight %}
- Concurrency control mechanism and transaction isolation can be disabled so that we can check out the state of database in the middle of a transaction from a database browser.<br/>
{% highlight yml %}
jdbc-url=jdbc:h2:mem:mydatabase;MVCC=FALSE;MV_STORE=FALSE;LOCK_MODE=0;
{% endhighlight %}
- H2 is able to work in compatibility-mode to support sql dialect of other common databases such as Postgres, Mysql, Oracle, etc.<br/>
{% highlight yml %}
jdbc-url=jdbc:h2:mem:mydatabase;MODE=PostgreSQL;
{% endhighlight %}

## Implementation (scala-play-slick)
I have prepared [a sample web application](https://github.com/nimatrueway/browsing-h2-inmemory-samples/tree/master/play-slick-mcwire) with `Scala` `Play-2.5.x` `Macwire-2.x` `Slick-3.1.x` `ScalaTest-2.2.x` `H2-1.4.x` which implements the proposed solution.

In the sample we have two entities `User` and `LoginLog` and two evolution SQLs that create tables (`Users`, `LoginLogs`) and initialize them such that *nima* logs into the system *5 times*. There is a single controller named `MainController` which has only one service `nimaLoginCount` that prints login times of user “nima”.

The only test class `MainSpec` is a single small test on `MainController` to check if `nimaLoginCount` returns “5”. Notice that the very first line of the test starts h2-server so that we can connect our database-browser tool (My favorite is `DBVisualizer`) to it.

{% highlight scala %}
class MainSpec extends PlaySpec {
  "nimaLoginCount() should return nima's loginCounts after all evolutions-scripts are applied" in {
    H2ServerBuilder.start // starts h2-server in the beginning
    val getStudentsFakeRequest = FakeRequest(routes.MainController.nimaLoginCount())
    val resultFuture = route(ApplicationBuilder.build, getStudentsFakeRequest).get
    status(resultFuture) mustEqual OK
    contentAsString(resultFuture) mustEqual "5"
  }
}
object H2ServerBuilder {
  def start = Server.createTcpServer ("-tcp", "-tcpAllowOthers", "-tcpPort", "9092", "-tcpDaemon").start()
}
{% endhighlight %}

Tests use `application.test.conf` configuration which basically includes `application.conf` and then overrides original postgres database-configuration with a H2 in-memory database, that is compatible to postgres sql-dialect, with concurrency and transaction-isolation disabled.

{% highlight yml %}
# Include original application configuration
include "application.conf"

# Override database configuration
slick.dbs.default.driver="slick.driver.H2Driver$"
slick.dbs.default.db.driver="org.h2.Driver"
slick.dbs.default.db.url="jdbc:h2:mem:mydatabase;MODE=PostgreSQL;MVCC=FALSE;MV_STORE=FALSE;LOCK_MODE=0;"
slick.dbs.default.db.user="sa"
slick.dbs.default.db.password="123"
{% endhighlight %}

Everything is ready for the action. So let’s put a breakpoint just before the sql query of `nimaLoginCount`, then proceed to debug the single test with IntelliJ.

![](/img/posts/001-screenshot1-debug.png)

Program runs, then hits the breakpoint. As you can see in the picture below, we can extract the exact sql to-be-run from slick's `query.result` like this: `queryResult.statements.head` and manually run it on database ourself.

![](/img/posts/001-screenshot2-breakpoint.png)

Great ! H2 server is up-and-running, the application has been suspended just before the query. Now it's time to run a database browser (in my case `DBVisualizer`), configure a database connection of type `h2-embedded` with a <u>jdbc-url that is basically the same jdbc-url of our application but</u> `tcp://localhost:9092/` <u>added before</u> `mem` :

{% highlight yml %}
jdbc:h2:tcp://localhost:9092/mem:mydatabase;MODE=PostgreSQL;MVCC=FALSE;MV_STORE=FALSE;LOCK_MODE=0;
{% endhighlight %}

![](/img/posts/001-screenshot3-dbvis-connect.png)

We can run the query on database ourselves :

![](/img/posts/001-screenshot4-dbvis-sql.png)

Now we got the result we were expecting. No need for further debugging !

## Implementation (java-spring-flyway)

There is [another sample](https://github.com/nimatrueway/browsing-h2-inmemory-samples/tree/master/springboot-flyway) that I have prepared using `Java` `SpringBoot-1.5.x` `Hibernate-5.x` `Flyway-4.0.x`. It's completely similar to the scala version. I encountered a couple of teeny-tiny problems that I think are worth mentioning, because you might bump into them too.

* When all connections to H2 in-memory database are disconnected, the database destroys itself by-default. There might be a very small gap between the moment **Flyway has applied the evolutions** and when **the service runs the query**, So if a connection-pool is not present, this small gap might cause the database to be re-created.
 We can avoid that by adding the parameter `DB_CLOSE_DELAY={seconds};` to jdbc-connection url which makes H2 wait for a while, before deciding to destroy the database.

{% highlight yml %}
spring.datasource.url=jdbc:h2:mem:mydatabase;DB_CLOSE_DELAY=10;MODE=PostgreSQL;MVCC=FALSE;MV_STORE=FALSE;LOCK_MODE=0;
{% endhighlight %}

* Hibernate's default physical naming strategy is `SpringPhysicalNamingStrategy` which automatically puts an underline `_` between subsequent Captial words which would disrupt my application (I have a table named `UserLogs`). Add this line to spring configuration properties file to fix it:

{% highlight yml %}
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
{% endhighlight %}

* H2 SQL dialect requires all table / field names to be quoted, otherwise it will tell you that such table / field does not exists ! When I wrote a HQL on one repository class, I figured hibernate does not quote anything and in turn H2 reports `table UserLogs does not exist !`. One simple solution was to quote names on their very definitions which solved my problem:
{% highlight java %}
@Entity
@Table(name = "`Users`")
public class User {
...
}
{% endhighlight %}


## Considerations and Warnings

* Make sure the breakpoints you put **suspend only one thread** and not all threads, as H2-Server runs on a thread itself and its suspension will **freeze any external connection to the database**.

![](/img/posts/001-screenshot5-breakpoint.png)

* Sometimes IPv4/IPv6 incompatibilities cause wired complications. For example your application's H2-Server might only bind to IPv6 (::1) but your DB Browser tries to connect to IPv4 (127.0.0.1) and they can not connect. You can always force your JVM to stick to IPv4 using JVM option `-Djava.net.preferIPv4Stack=true`.

* The version of H2 driver your application is using must be the same of H2 driver your DB Browser uses (or at least they are compatible) otherwise you will get unknown exception while trying to connect DB Browser to your application. (if you are using `DBVisualizer` just copy `h2.jar` from `.m2` / `.ivy` repository to `{db-vis}/jdbc/h2/h2.jar`")

* After fixing the single test, remember to re-enable concurrency control and transaction isolation ( **remove** `MVCC=FALSE;MV_STORE=FALSE;LOCK_MODE=0;`) to prevent confliction between the tests that concurrently access a row, especially if you are using database for several tests that run in parallel, .

*   Having a debugger is **not mandatory** at all. Any mechanism that can pause the program at a certain point in the service is sufficient. It could be as simple as:<br/>
    `JOptionPane.showMessageDialog(null, "Service has been paused !")`

* If you are using a test-kit that executes assertion on async-results of service. it probably has a certain timeout which would cause the test to fail and application to terminate after a certain while. In my case (`play-test`) there is a `Helpers` object (in package `play.api.test`) that has different methods for making assertion on async-result of a play web service. This object extends `DefaultAwaitTimeout` which defines timeout to be 20 seconds by-default: <br/>`implicit def defaultAwaitTimeout: Timeout = 20.seconds`<br/>
  We would like to keep the test running, while application is paused and we are browsing H2 database. So we can created an object `TestHelpers` just like the original `Helper` object but override the defaultAwaitTimeout with a very long timeout value and use it instead :

{% highlight scala %}
object TestHelpers extends PlayRunners
  with HeaderNames
  with Status
  with MimeTypes
  with HttpProtocol
  with DefaultAwaitTimeout
  with ResultExtractors
  with Writeables
  with EssentialActionCaller
  with RouteInvokers
  with FutureAwaits {
  override implicit def defaultAwaitTimeout = 1 day
}
{% endhighlight %}

## Links

1. [Two sample projects implementing this idea](https://github.com/nimatrueway/browsing-h2-inmemory-samples)