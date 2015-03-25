# CQRS Blog #

Just a few notes while I'm going through a Greg Young video on Command Query Responsibility Separation (CQRS) and Event Sourcing.

A typical app has a storage layer, some DTOs and an app service layer...

In code then, the service layer might look like the following:

```csharp
public class InventoryService {
  IInventoryStore store;

  public InventoryService(IInventoryStore store){
    this.store = store;
  }

  public void DeactivateInventoryItem(int id, string reason){
    var item = this.store.Get(id);
    item.Deactivate(reason);
    this.store.Update(item);
  }
}
```

To get to a CQRS model we'll introduce the following topics in order:

1. [Commands](./commands)
2. [Command Handlers](./command-handlers)
3. [Events](./events)
4. [Event Store](./event-store)
5. [Aggregates](./aggregates)
6. [Event Sourcing](./event-sourcing)
