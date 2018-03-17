# Transactions

  
All reads and writes to the database should be wrapped in a transaction to ensure that the changes are [ACID](http://en.wikipedia.org/wiki/ACID). 

## Achieving ACID Compliance

### Atomicity

As defined on [Wikipedia](https://en.wikipedia.org/wiki/Atomicity_%28database_systems%29), an atomic transaction "is an indivisible and irreducible series of database operations such that either all occur, or nothing occurs." Starcounter ensures atomicity by wrapping changes of one transaction within a transaction scope. The changes commit simultaneously at the end of the scope. If something interrupts the transaction before the end of the scope is reached, none of the changes will commit to the database.

### Consistency

A consistent DBMS ensures that all the data written to the database follow the defined contraints of the database. In Starcounter, this is solved by raising exceptions when an "illegal" action is carried out within a transaction, such as commiting non-unique values to a field that requires unique values. The exception will in turn make the transaction roll back so that none of the changes are applied.

### Isolation

To make transaction isolated, Starcounter uses [snapshot isolation](https://en.wikipedia.org/wiki/Snapshot_isolation). This means that when a transaction initializes, it takes a snapshot of the database and stores it in a transactional memory. This means that every transaction sees its own snapshot of the database. For example, a SQL query that executes before a transaction commits will not be able to see the changes made by the transaction because the changes are isolated to that transaction's snapshot of the database. This works no matter how large the database is.

### Durability

Durability ensures that commited transactions will survive permanently. Starcounter solves this by writing transactions to a transaction log after commits.

### Concurrency Control

When many users write to the database at the same time, the database engine must ensure that the data keeps consistent using atomicity and isolation. For example, if an account reads 100 and you want to update it to 110 and another transaction is simultaneously reading a 100 and wants to update it to 120. Should the result be 110, 120 or 130? To resolve this, the transaction must be able to handle conflicts. The easiest way to do this is to use locking. If you want your database engine to serve large numbers of users and transactions, locking is slow and expensive and can lead to [deadlocks](http://en.wikipedia.org/wiki/Deadlock). Locking is efficient when you almost expect a conflict, i.e. when the probability is high that you will have a conflict. The slow nature of locking is that it always consumes time, even if there is no conflict. Another word for locking is 'pessimistic concurrency control'. A more efficient way of providing concurrency is to use 'optimistic concurrency control'. As the name implies, you don't expect a conflict, but you will still handle it. The concurrency control in Starcounter is **optimistic concurrency control**. It makes the assumption that conflicts between transactions are unlikely. The database can allow transactions to execute without locking the modified objects. If a conflict occurs, Starcounter will restart the transaction until it commits, or 100 times. For long-running transactions, the developer has to implement the retry functionality.

## Db.Transact

`Db.Transact` is the simplest way to create a transaction in Starcounter. It declares a transactional scope and runs synchronously, as described [above](transactions.md#achieving-acid-compliance). The argument passed to the `Db.Transact` method is a delegate containing the code to run within the transaction. In code, it looks like this:

```csharp
Db.Transact(() =>
{
    var person = Db.Insert<Person>();
    person.Name = "Gandalf";
});
```

  
Since `Db.Transact` is synchronous, it sometimes becomes a performance bottleneck. Starcounter handles this by automatically scaling the number of working threads to continue processing requests even if some handlers are blocked. The maximum number of working threads is the number of CPU cores multiplied by 254, so with four cores, there would be a maximum of 1016 working threads. When these threads are occupied, the next `Db.Transact` call in line will have to wait until a thread is freed. [Db.TransactAsync](transactions.md#db.transactasync) circumvents this.

`Db.Transact` is an implementation of `Db.TransactAsync` with a thin wrapper that synchronously waits for the returned `Task` object.

## Db.TransactAsync

`Db.TransactAsync` is the asynchronous counterpart of `Db.Transact`. It gives the developer more control to balance throughput and latency. The function returns a `Task` that is marked as completed when flushing the transaction log for the transaction. Thus, the database operation itself is synchronous while flushing to the transaction log is asynchronous.

 `Db.Transact` and `Db.TransactAsync` are syntactically identical:

```csharp
Db.TransactAsync(() => 
{
    // The code to run in the transaction
})
```

While waiting for the write to the transaction log to finish, it's possible to do other things, such as sending an email:

```csharp
Order order = null;
Task task = Db.TransactAsync(() =>
{
    order = new Order();
}); // Order has been added to the database

SendConfirmationEmail(order);

task.Wait(); // Wait for write to log to finish
```

This is more flexible and performant than `Db.Transact`, but it comes with certain risks; for example, if there's a power outage or other hardware failure after the email is sent but before writing to the log, the email will be incorrect - even if the user got a confirmation, the order will not be in the database since it was never written to the transaction log. 

If a handler creates write transactions, use `Db.TransactAsync` and then wait for all transactions at once with `Task.WaitForAll` or `TaskFactory.ContinueWhenAll`. Otherwise, the latency of the handler will degrade.

### Limitations

With the `Db.TransactAsync` API, it's tempting to use async/await in applications. Syntactically it's possible, although it's not that useful due to these limitations:

1.  Using async/await is not possible in a handler body as Handler API doesn't support async handlers.
2.  No special measures have been taken to force after-await code to run on Starcounter threads, so manual `Scheduling.ScheduleTask` might be required \(see [Running background jobs](running-background-jobs.md) for details\).
3.  use async/await with caution as they may inadvertently increase the latency. Say if user code runs transactions sequentially, putting await in front of every `Db.TransactAsync` will accumulate all the individual latencies. The right stategy in this case is to make a list of tasks and then await them at once.

## Nested Transactions

Transactions can't be nested unless you specify what to do on commit for the inner transaction. To understand why it's this way, take a look at this example:

```csharp
public void OrderProduct(long productId, Customer customer)
{
    Db.Transact(() =>
    {
        SendInvoice(productId, customer);
        RemoveFromInventory(productId);
    });
}

public void SendInvoice(long productId, Customer customer)
{
    var invoice = new Invoice(productId, customer);
    Db.Transact(() => AddInvoiceToDb(invoice));
    invoice.Send();
}
```

After the transaction on line 13, you'd expect that the invoice has been committed to the database. That is not the case. To preserve the atomicity of the outer transaction, the changes have to be committed at the same time at the end of the outer transactions' scope. Thus, if the invoice was sent on line 14 and then the whole transaction could roll back in `RemoveFromInventory` and undo `AddInvoiceToDb`. This would cause the customer to have received an invoice that is not stored in the database.  

Due to this, inner transactions have to specify what to do on commit. To adapt the previous example to specify what to do on commit, we would do this to `SendInvoice`:

```csharp
public void SendInvoice(long productId, Customer customer)
{
    var invoice = new Invoice(productId, customer);
    Db.Transact(() => AddInvoiceToDb(invoice), 
        new TransactOptions(() => invoice.Send())); 
}
```

With this change, the invoice would be sent whenever the changes in `AddInvoiceToDb` are committed and you can be sure that the invoice would be sent first when the invoice is safely in the database. In this case, it would be when the outer transaction scope terminates. If there was no outer transaction, the invoice would be sent when the transaction with `AddInvoiceToDb` terminates. Thus, `onCommit` ensures that the calls are made in the correct order no matter what.  
  
If you don't know if a transaction will be nested within another transaction, it's always safe to add an empty `onCommit` delegate:

```csharp
Db.Transact(() => 
{
    // Read and write to database
}, new TransactOptions(() => {}));
```

With an empty `onCommit` delegate, you acknowledge that the changes are not guaranteed to commit after the transaction scope. 

If an inner transaction doesn't have an `onCommit` delegate, Starcounter will throw `ArgumentNullException`. 

## Transaction Size

Code in `Db.Transact` and `Db.TransactAsync` should execute in as short time as possible because conflicts are likelier the longer the transaction is. Conflicts requires long transactions to run more times which can be expensive. The solution is to break the transaction into smaller transactions.

## Exceptions

### ScErrReadOnlyTransaction

Reads or writes to the database without a transaction with throw an exception:

```text
The transaction is readonly and cannot be changed to write-mode. (ScErrReadOnlyTransaction (SCERR4093))
```

 For example:

```csharp
[Database]
public abstract class Person {}

class Program
{
    static void Main()
    {
        Db.Insert<Person>(); // SCERR4093
    }
}
```

 To fix this, wrap the operation in a transaction:

```csharp
[Database]
public abstract class Person {}

class Program
{
    static void Main()
    {
        Db.Transact(() => 
        {
            Db.Insert<Person>();
        })
    }
}
```

