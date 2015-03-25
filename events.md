# Events #

The next step is to replace the code in the event handler so that rather than updating our data store directly it instead generates an event.


```csharp
public class InventoryItemDeactivated: Event {
  public readonly int id;
  public readonly string reason;

  public InventoryItemDeactivated(int id, string reason)
  {
    this.id = id;
    this.reason = reason;
  }
}

public class DeactivateInventoryHandler: Handles<DeactivateInventoryItem> {
  IEventStore store;
  IEventBus bus;

  public InventoryService(IEventStore store, IEventBus bus){
    this.store = store;
    this.bus = bus;
  }

  public void Handle(DeactivateInventoryItem command){
    // Check to see if the command is valid
    ...

    // Alright - effect the command... this consists of writing
    // an event to the event store and then pushing the event
    // up to the event bus for other projections.
    var fact = new InventoryItemDeactivated(
      command.id, command.reason
    );
    store.add(fact);
    bus.add(fact);
  }
}
```

The event has exactly the same structure as the command. The only difference is that a command is a request to do something and it can be rejected. If the command succeeds, it generates an event - which is something that has already happened and it **cannot be changed**. You can't edit events. You can't delete them. You can't cancel them... They are facts.

The event will be stored in an event store, which is the single source of truth for what happens in our application. From the event store we can also then generate different projections for various different purposes (one of which might be a 3rd normal form domain model, maybe another which is a first normal form appropriate for reporting and maybe others again for other weird scenarios).
