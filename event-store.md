# Event Store #

Obviously we need somewher to store our events. As a bare minimum we will need a couple of tables:

```
AGGREGATES
AggregateId       Guid
Type              Varchar
CurrentVersion    Int
```
```
AGGREGATEEVENTS
AggregateId   Guid
Data          Blob
Version       Int
```

And some pseudo-code for the stored proc to create one or more events might be:

```sql
select currentVersion from AGGREGATES where AggregateId = @aggregateId
if version = 0
  insert into AGGREGATES ...
  version = 0
end
if @expectedVersion != currentVersion
  raise concurrency problem
end
foreach event
  insert event with incremented version number  
update aggregate with latest version number
```

Also we'd probably want to add a snapshot entity:

```
AGGREGATESNAPSHOTS
AggregateId     Guid
SerializedData  Blob
Version         Int
```

## Third party event stores ##

For the most part Greg Young advocates building a CQRS system yourself from scratch - with the exception of the event store. For this, he strongly advises piggy backing off an existing project:

* The most popular in .NET land is Jonathan Oliver's [NEventStore](https://github.com/NEventStore/NEventStore)
* Greg Young has created a platform agnostic [eventstore](https://geteventstore.com/)
