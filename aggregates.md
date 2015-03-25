# Aggregates #

Probably the most significant change is to the domain model. Whilst the read model (on the request side of our architecture) still contains bare bones DTO objects that are essentially just a list of properties to transfer over the wire, the bulk of our logic shifts from service classes into some new domain objects called Aggregate Roots.

An aggregate root is essentially a list of events. Current state (or indeed state at any given time) can be inferred simply by *applying* each of the events in the aggregate one after the other.

```csharp
public class InventoryItem : AggregateRoot
{
    private bool _activated;
    private Guid _id;

    private void Apply(InventoryItemCreated e)
    {
        _id = e.Id;
        _activated = true;
    }

    private void Apply(InventoryItemDeactivated e)
    {
        _activated = false;
    }

    public void Deactivate()
    {
        if(!_activated) throw new InvalidOperationException("already deactivated");
        ApplyChange(new InventoryItemDeactivated(_id));
    }

    public override Guid Id
    {
        get { return _id; }
    }

    public InventoryItem()
    {
    }

    public InventoryItem(Guid id, string name)
    {
        ApplyChange(new InventoryItemCreated(id, name));
    }
}

```

This is a really simple aggregate with just one property (other than the id) and two commands (create and deactivate) that can be performed on it... but serves to illustrate how a more complex aggregate might be built.

The following is the `AggregateRoot` class that this inherits from:

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

    protected void ApplyChange(Event @event)
    {
        ApplyChange(@event, true);
    }

    // push atomic aggregate changes to local history for further processing (EventStore.SaveEvents)
    private void ApplyChange(Event @event, bool isNew)
    {
        this.AsDynamic().Apply(@event);
        if (isNew) _changes.Add(@event);
    }
}

```

There are some really important things to note about this code:

## Apply methods contain no logic ##

The apply methods **never contain logic** and **never raise exceptions**. So no parameter checking ever happens in an apply method. The reason for this is quite simple: they are applying events which are facts - things that have happened in the past. You can't change those events.

## Apply methods are always private ##

The only members that should need to access the apply methods are the public "Action" methods (which get called by command handlers) and a `LoadFromHistory` method, which I'd describe in the next section on [event sourcing](./event-sourcing).

## public methods take commands and generate events

Command handlers will call public methods like the constructor for the aggregate (which implicitly creates an "item created" event) and the `Deactivate()` method in our example code. These public methods do stuff like making sure that it's OK to effect that change being requested. If it's not, they can raise an exception.

If no exception is raised however, the public methods will effect the command by adding a new event to the aggregate. Once this happens we no longer have a request - we have an immutable fact and no more exceptions can be raised (assuming there are no concurrency issues when persisting the event back to our event store).

These public methods are therefore the tipping point at which commands become events.
