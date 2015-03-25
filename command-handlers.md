# Command Handlers #

Generally, at that stage, we no longer have a *service* in the traditional sense of the word so we can rename this to be a command handler and add an interface to indicate what kinds of commands it can handle:

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

public interface Handles<T> where T: Command {
  void Handle(T command);
}

public class DeactivateInventoryHandler: Handles<DeactivateInventoryItem> {
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

Greg comments at this stage that it is now pretty trivial to add stuff like logging or authorisation to our application. Our commands are now serialisable. Indeed, we can add all sorts of aspect oriented like stuff simply by chaining together event handlers.
