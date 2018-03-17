# Database

In Starcounter, classes are tables and class instances are rows. The database objects live in the database from the beginning. This means that they are not serialized to the database, they are _created_ in the database from the time you use the new operator. SQL queries will immediately see them. There is no concept of moving data to and from the database. This means that accessing a property on a database object reads the value from the database rather than from the .NET heap. This is possible as the data of the database lives in the RAM.

## Creating the Database

The database is created explicitly in code by calling `ScCreateDb.Execute` which is found in the `Starcounter.Core.Bluestar` namespace. `ScCreateDb.Execute` takes the name of the directory where the database files should be created and an optional parameter for specifying the name of the database. If no name is provided, the name of the directory that the database files are in will be taken as the database name. 

Calling `ScCreateDb.Execute` with the same directory name twice will throw an exception. To deal with this, check if there is a database first, and if there's not, call `ScCreateDb.Execute` . This can be done with `StarcounterOptions.TryOpenExisting` which is found in the `Starcounter.Core.Options` namespace:

```csharp
const string databaseName = "myDatabase";

if (!Starcounter.Core.Options.StarcounterOptions.TryOpenExisting(databaseName))
{
    System.IO.Directory.CreateDirectory(databaseName);
    Starcounter.Core.Bluestar.ScCreateDb.Execute(databaseName);
}
```

This will create a new directory with the specified database name and create a database with the same name only if there's no database with that name. 

  
 

