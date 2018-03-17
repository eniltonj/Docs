# Threading

`Starcounter.Core` enforces thread-safe database access.

The database kernel is not in itself thread-safe for performance reasons, because codehosts can have very different thread models, from single-threaded Javascript, C\# pool of worker threads to Go's millions of goroutines.

As an application developer, you can expect a `Transaction` to be accessible from one thread at a time. If you try to access it from multiple threads in parallel, `Starcounter.Core` will attempt to serialize the accesses. If that is not possible, either `Transaction.Current` will be `null` or your application will deadlock.

Instead of using multiple threads sharing single transaction \(which would act as a mutex\), use the `async/await` pattern. That will allow the .NET runtime and `Starcounter.Core` to use resources as efficiently as possible.

This won't allow you to parallelize the execution since the underlying database operation won't allow it, but it will allow the .NET runtime to optimize worker thread usage, keeping the overall number of threads as low as possible.

For more information on the Task-based Asynchronous Pattern \(TAP\), see [https://msdn.microsoft.com/en-us/library/hh873175\(v=vs.110\).aspx](https://msdn.microsoft.com/en-us/library/hh873175%28v=vs.110%29.aspx)

```csharp
// Using await here allows this worker thread to do other
// work while we wait for the transaction to execute.
// Note that if the transaction is expected to be quick,
// it's more efficient to use the synchronous Db.Transact().
await Db.TransactAsync(() => 
{
    var person = Db.Insert<Person>();
    person.FirstName = "John";
    person.LastName = "Smith";
    person.FindFriends(); // Takes a long time
});
```

Discussion thread: [https://github.com/Starcounter/Starcounter.Core/issues/78](https://github.com/Starcounter/Starcounter.Core/issues/78)

