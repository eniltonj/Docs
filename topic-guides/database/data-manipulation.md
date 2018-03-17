# Data Manipulation

Data manipulation is mainly done in C\# code.

Reads and writes to the database have to be wrapped in a [transaction.](transactions.md)

## Creating Database Objects

Database objects are created with `Db.Insert`:

```csharp
Db.Insert<Person>();
```

The created object is returned from the method so the properties can be set:

```csharp
var person = Db.Insert<Person>();
person.Name = "John";
```

Trying to create database objects with `new` will fail to store the object persistently.

Creating objects `Db.Insert` will call their non-private constructors as described in under [creating database classes](database-classes.md#constructors).

## Update

To update a database object, retrieve it with `Db.SQL` and update with the native program code assign operator `=`.

For example, to update the `LastName` of all the `Person` objects in the database, they would be looped through and updated:

```csharp
var people = Db.SQL<Person>("SELECT p FROM Person p");
foreach (var person in people)
{
    person.LastName = person.LastName.ToUpper();
}
```

## Delete

There are two ways to delete database objects:

1. Using the `Delete` method on an object
2. Using `DELETE FROM`

`Delete` is used for single objects and `DELETE FROM` is used for many objects.

They look like this:

```csharp
var john = Db.Insert<Person>();
john.Delete();

Db.SQL("DELETE FROM Person");

Db.SQL("DELETE FROM Person WHERE Name = ?", "John");
```

`person.Delete()` will just delete `john` while `DELETE FROM Person` will delete all objects of the `Person` class.

