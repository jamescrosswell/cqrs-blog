# Commands #

The first step to converting this to a CQRS architecture is to factor out the parameters of the `AddInventory` method into a command object:

```csharp
public class DeactivateInventoryItem: Command {
  public readonly int id;
  public readonly string reason;

  public DeactivateInventoryItem(int id, string reason)
  {
    this.id = id;
    this.reason = reason;
  }
}

public class InventoryService {
  IInventoryStore store;

  public InventoryService(IInventoryStore store){
    this.store = store;
  }

  public void Handle(DeactivateInventoryItem command){
    var item = this.store.Get(command.id);
    item.Deactivate(command.reason);
    this.store.Update(item);
  }
}
```
