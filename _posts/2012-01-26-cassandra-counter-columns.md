---
layout: post
title: Cassandra Counter Columns
tags:
- Cassandra
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Cassandra supplies "counter columns", which are used to store a number that incrementally counts a value. You might use a counter to keep track of pageviews or other events. 

I couldn't find much documentation about how to use these, especially through the CLI (which is where I typically start out when trying to investigate a new feature). So here goes with an explanation of what I've found.

Create your "counter" column family:

    create column family MyCounters with default_validation_class=CounterColumnType and comparator=UTF8Type;

Note that you don't _have_ to create the column family with a "default\_validation\_class" of CounterColumnType, but by doing so you get a few major advantages:

1. You can create arbitrary column names as counters, rather than having to add them explicitly and being required to know ahead of time what you'll be counting
2. I couldn't figure out how to actually do that, despite the fact that the documentation I read seemed to indicate it should be possible; instead, when I attempted to **update column family** with some column\_metadata, it failed, like so:

        [default@MyTest] update column family MyCounters with column_metadata = [{column_name:thing-1, validation_class:CounterColumnType}];
        org.apache.thrift.TApplicationException: Internal error processing system_update_column_family

That's all the information you get in the CLI. Apparently, when you see "Internal error", that means you need to look in **/var/log/cassandra/system.log**, where you'll see:

    ERROR [pool-2-thread-1] 2012-01-26 09:41:01,705 Cassandra.java (line 4038) Internal error processing sys
    tem_update_column_family
    java.lang.RuntimeException: org.apache.cassandra.config.ConfigurationException: Cannot add a counter col
    umn (thing-1) in a non counter column family
            at org.apache.cassandra.thrift.CassandraServer.system_update_column_family(CassandraServer.java:
    1049)
            at org.apache.cassandra.thrift.Cassandra$Processor$system_update_column_family.process(Cassandra
    .java:4032)
            at org.apache.cassandra.thrift.Cassandra$Processor.process(Cassandra.java:2889)
            at org.apache.cassandra.thrift.CustomTThreadPoolServer$WorkerProcess.run(CustomTThreadPoolServer
    .java:187)
            at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
            at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
            at java.lang.Thread.run(Thread.java:680)
    Caused by: org.apache.cassandra.config.ConfigurationException: Cannot add a counter column (thing-1) in a
     non counter column family
            at org.apache.cassandra.config.CFMetaData.validate(CFMetaData.java:994)
            at org.apache.cassandra.config.CFMetaData.fromThrift(CFMetaData.java:684)
            at org.apache.cassandra.thrift.CassandraServer.system_update_column_family(CassandraServer.java:
    1045)
            ... 6 more

So that's why it's important to **create column family** as a "counter column family" from the start. Because of this, I intend to use these counter column families as what are essentially extra, metadata-only denormalized column families; ie, I won't be mixing actual data in with these counters. Instead, the actual data will live elsewhere, and the counters will live apart, by themselves. Remember, in Cassandra, this sort of denormalization is considered not only acceptable, but required.

Okay, enough of that diversion. Let's continue with using the CLI to make and use our counter column family.

I usually tell the CLI to use UTF-8, rather than raw hex bytes (I don't know why it defaults to expecting you, a human, to be able to read and write raw bytes):

    assume MyCounters keys as utf8;

Now you can use **incr** to increment your counters:

    incr MyCounters['key-1']['thing-1'];
    incr MyCounters['key-1']['thing-2'];
    incr MyCounters['key-1']['thing-1'];
    incr MyCounters['key-2']['thing-1'];
    incr MyCounters['key-2']['thing-3'];
    incr MyCounters['key-2']['thing-3'];
    incr MyCounters['key-1']['thing-4'] by 5;

And you can pull out the whole thing with **list**:

    [default@MyTest] list MyCounters;
    Using default limit of 100
    -------------------
    RowKey: key-1
    => (counter=thing-1, value=2)
    => (counter=thing-2, value=1)
    => (counter=thing-4, value=5)
    -------------------
    RowKey: key-2
    => (counter=thing-1, value=1)
    => (counter=thing-3, value=2)

    2 Rows Returned.
    Elapsed time: 8 msec(s).

Or you can **get** the values for a single RowKey:

    [default@MyTest] get MyCounters['key-1'];
    => (counter=thing-1, value=2)
    => (counter=thing-2, value=1)
    => (counter=thing-4, value=5)
    Returned 3 results.
    Elapsed time: 8 msec(s).

You can also **get** the value of a single counter within a column:

    [default@MyTest] get MyCounters['key-1']['thing-1'];
    => (counter=thing-1, value=2)
    Elapsed time: 8 msec(s).

As you can see, counter columns are not actually complicated or difficult to use. This got me around some of the initial issues I encountered, so hopefully it helps someone else.

**One last thing**

When using counters, there are [some things to consider](http://www.datastax.com/docs/1.0/ddl/column_family#about-counter-columns) when it comes to your consistency level:

> it’s important to understand that unlike normal columns, a write to a counter requires a read in the background to ensure that distributed counter values remain consistent across replicas. If you write at a consistency level of ONE, the implicit read will not impact write latency, hence, ONE is the most common consistency level to use with counters.

I haven't addressed this yet, but I'll keep it in mind.
