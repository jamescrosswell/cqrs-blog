# Event Sourcing #

Once we have the concept of aggregates, moving to event sourcing is dead easy. We simply need to add a `LoadFromHistory` method to our `AggregateRoot` class. Then, rather than reading the "current state" of an object from a 3rd normal form database we can fetch all of the events for the aggregate that represents that object and then simply replay them (i.e. apply each of them in order). The result will be an aggregate root that represents the *current state* of the object that that aggregate models. 

```csharp
public abstract class AggregateRoot
{
    private readonly List<Event> _changes = new List<Event>();

    public abstract Guid Id { get; }
    public int Version { get; internal set; }

    public IEnumerable<Event> GetUncommittedChanges()
    {
        return _changes;
    }

    public void MarkChangesAsCommitted()
    {
        _changes.Clear();
    }

    public void LoadFromHistory(IEnumerable<Event> history)
    {
        foreach (var e in history) ApplyChange(e, false);
    }

    protected void ApplyChange(Event @event)
    {
        ApplyChange(@event, true);
    }

    // push atomic aggregate changes to local history for further processing (EventStore.SaveEvents)
    private void ApplyChange(Event @event, bool isNew)
    {
        this.AsDynamic().Apply(@event);
        if(isNew) _changes.Add(@event);
    }
}

public interface IRepository<T> where T : AggregateRoot, new()
{
    void Save(AggregateRoot aggregate, int expectedVersion);
    T GetById(Guid id);
}

public class Repository<T> : IRepository<T> where T: AggregateRoot, new() //shortcut you can do as you see fit with new()
{
    private readonly IEventStore _storage;

    public Repository(IEventStore storage)
    {
        _storage = storage;
    }

    public void Save(AggregateRoot aggregate, int expectedVersion)
    {
        _storage.SaveEvents(aggregate.Id, aggregate.GetUncommittedChanges(), expectedVersion);
    }

    public T GetById(Guid id)
    {
        var obj = new T();//lots of ways to do this
        var e = _storage.GetEventsForAggregate(id);
        obj.LoadFromHistory(e);
        return obj;
    }
}
```
