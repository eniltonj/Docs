# Querying Using SQL

SQL queries are executed using the `Db.SQL` method. If the SQL command is `SELECT`, the method returns `SqlResult<T> : IEnumerable<T>`, it otherwise returns `null`.

```csharp
Db.SQL("SELECT p FROM Person p"); // => SqlResult<Person>
Db.SQL("SELECT p.Name FROM Person p"); // => SqlResult<string>
Db.SQL("DELETE FROM Person"); // => null
```

In addition to traditional SQL, Starcounter allows you to select objects in addition to primitive types such as strings and numbers. Also it allows you to use C\# style path expressions such as `person.FullName`.

{% hint style="warning" %}
When querying with `Db.SQL`, there are [certain reserved words](../../reference/sql-old/#reserved-words) that should be escaped by surrounding them in quotation marks.
{% endhint %}

See more about SQL in [SQL](../../reference/sql-old/).

## SQL Error

If a query cannot be processed due to some syntax or type checking error then the method `Db.SQL` will throw the `SqlException` `ScErrSQLIncorrectSyntax (SCERR7021)`.

```csharp
try
{  
    var people = Db.SQL<Person>(
        "SELECT e.NonExistingProperty FROM Person p");
    
    foreach(Person person in people)  
    {    
        Console.WriteLine(person.Name);  
    }
}
catch (SqlException exception)
{  
    Console.WriteLine("Incorrect query: " + exception.Message);
}
```

## Variables

In programming code you are not allowed to use literals in SQL queries. Instead you should use SQL variables in the query string and pass the current values of the variables as parameters when executing the query. The reason is to achieve better performance and to save system resources.

SQL variables are represented by question marks \(?\) in the query string, and you pass the current values of the variables as parameters to the method `Db.SQL(String query, params Object[] values)`.

```csharp
var employees = Db.SQL<Employee>("SELECT e FROM Employee e WHERE e.FirstName = ?", "Joe");
foreach (var employee in employees)
{
  Console.WriteLine($"{employee.FirstName} {employee.LastName}");
}
```

You can pass an arbitrary number of variable values to the method SQL, but the number of variables values needs to be exactly the same as the number of variables \(question marks\) in the query string. Otherwise, an `ArgumentException` will be thrown.

Each variable will have an implicit type depending on its context in the query string. For example a variable that is compared with a property of type String will implicitly be of type String. If a variable is given a value of some incompatible type then an `InvalidCastException` will be thrown. All numerical types are compatible to each other.

Note that you can only use ? to variables after the WHERE clause. You cannot, for instance, use ? to replace the class name of a query.

## Changing Query Processor

Starcounter comes with two query processors - an old one and a new one. The new one supports a larger set of SQL than the old query processor. Although, the old one still supports some features that the new one doesn't yet have. To give you access as to as many features as possible, you can switch between the two.

By default, the old query processor is enabled. You can verify this by running `Db.GetDefaultQueryProcessorName()`. If it returns "Level1Bridge", the old query processor is enabled by default. If it returns "Experimental", the new query processor is enabled.

To change query processor, use `TrySetCurrentQueryProcessor`:

```csharp
Console.WriteLine(Db.GetDefaultQueryProcessorName()); // => Level1Bridge

Db.TrySetCurrentQueryProcessor("EXPERIMENTAL");
Console.WriteLine(Db.GetCurrentQueryProcessor().Name); // => Experimental

Db.TrySetCurrentQueryProcessor("Level1Bridge");
Console.WriteLine(Db.GetCurrentQueryProcessor().Name); // => Level1Bridge
```

The old query processor supports the SQL documented in the [old SQL reference](../../reference/sql-old/).

The new query processor supports the SQL documented in the [new SQL reference](../../reference/sql-new/).

