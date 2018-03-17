# Database Classes

## Introduction

The database schema in Starcounter is defined by C\# classes with the `[Database]` attribute:

```csharp
[Database]
public abstract class Person 
{
    public abstract string FirstName { get; set; }
    public abstract string LastName { get; set; }
    public string FullName => $"{FirstName} {LastName}";
} 
```

All instances of database classes are stored persistently and can be queried for with SQL. 

## Class Declaration

Database classes have to be `public` . Creating instances of database classes that are not `public` will throw `ScErrTypeNotFound (SCERR15003)` . 

To simplify development, we recommend marking classes as `abstract`. This will give you a compilation error when trying to create a new objects with the `new` keyword. Instances of database classes created with `new` will not be added to the database, so `abstract` is used to avoid having code silently fail to add objects to the database. Read more in [Creating Database Objects](database-classes.md#creating-database-objects).

## Constructors

Database classes support constructors without parameters. Non-private constructors are called when a new instance is created with `Db.Insert`,  private constructors are ignored by Starcounter and will not be called on instantiation. 

If you create a constructor with parameters on a database class, it will throw `System.NotSupportedException`.

For example, this is a valid database class with a constructor:

```csharp
[Database]
public abstract class Order
{
    public Order()
    {
        this.Created = DateTime.Now;
    }

    public abstract DateTime Created { get; set; }
}
```

{% hint style="warning" %}
All C\# access modifiers are accepted for constructors, except for `internal`. Using it will throw `ScErrSchemaCodeMismatch (SCERR4177)`
{% endhint %}

## Fields and Properties

Database classes should only use properties - either auto-implemented or with an explicitly declared body. 

Properties should also be `public` and either `virtual` or `abstract` . 

### Collections

It is possible to have collections in the database class if the collection has an explicitly declared body. For example, the following properties are allowed:

```csharp
public List<string> Branches => 
    new List<string>(){ "develop", "master" };

public IEnumerable<Person> Friends => 
    Db.SQL<Person>("SELECT p FROM Person p");
```

These properties and fields are not allowed:

```csharp
public string[] Names { get; set; }
public List<Person> People { get; }
public IEnumerable Animals;
```

To access collections from database objectss, first retrieve the object and then access the property that has the collection:

```csharp
var person = Db.SQL<Person>("SELECT p FROM Person p").FirstOrDefault();
IEnumerable<Person> friends = person.Friends;
```

## Create Index on Property

An index can be created on a property with the index attribute:

```csharp
[Database]
public abstract class Person 
{
    [Index]
    public abstract string FirstName { get; set; }
    public abstract string LastName { get; set; }
} 
```

## Limitations

### Property Limit

Database classes can have a maximum of 112 properties for performance reasons. Thus, this is not allowed:

```csharp
[Database]
public abstract class LargeClass
{
    public abstract string Property1 { get; set; }
    public abstract string Property2 { get; set; }
    // ...
    public abstract string Property113 { get; set; }
}
```

If you have more than 113 properties, Starcounter will throw `ScErrToManyAttributes (SCERR4013)`

### Nested Classes

Nested database classes are not supported. The limitation is that inner database classes cannot be queried with SQL.

## Relations

### One-to-Many Relations

We recommend modeling one-to-many relationships by having references both ways - the child has a reference to the parent and the parent has a reference to all the children.

In this example there is a one-to-many relationship between`Department` and `Employee`:

```csharp
[Database]
public abstract class Department
{
  public IEnumerable Employees => 
    Db.SQL<Employee>(
      "SELECT e FROM Employee e WHERE e.Department = ?", this);
}

[Database]
public abstract class Employee
{
  public abstract Department Department { get; set; }
}
```

### Many-to-Many Relations

We recommend modeling many-to-many relationships with an associative class.

In this example there is a many-to-many relation between`Person` and `Company`- to represent this many-to-many relationship we use the associative class `Shares`:

```csharp
[Database]
public abstract class Person
{
  public IEnumerable EquityPortfolio => 
    Db.SQL<Shares>(
      "SELECT s.Equity FROM Shares s WHERE s.Owner = ?", this);
}

[Database]
public abstract class Company
{
  public IEnumerable ShareHolders => 
    Db.SQL<Shares>(
      "SELECT s.Owner FROM Shares s WHERE s.Equity = ?", this);
}

[Database]
public abstract class Shares
{
  public abstract Person Owner { get; set; }
  public abstract Company Equity { get; set; }
  public abstract int Quantity { get; set; }
}
```

## Inheritance

Any database object can inherit from any other database object.

```csharp
[Database]
public class Customer
{
   public string Name { get; set; }
}

public class PrivateCustomer : Customer
{
   public string Gender { get; set; }
}

public class CorporateCustomer : Customer
{
   public string VatNumber { get; set; }
}
```

The `Database` attribute is inherited from base- to subclasses. Any class that directly or indirectly inherits a class with the `Database` attribute becomes a database class. In the example above, both `PrivateCustomer` and `CorporateCustomer` become database classes due to them inheriting `Customer`.

The table `Customer` will contain all `PrivateCustomers` and all `CorporateCustomers`. So if there is a private customer called "Goldman, Carl" and a corporate customer called "Goldman Sachs", the result of `SELECT C FROM Customer c` will contain both of them.

### Base Classes

A base class contains all instances of all derived classes in addition to the instances with the its own exact type.

```sql
SELECT C FROM Customer C WHERE Name LIKE 'Goldman%'
```

Returns `[ { Name:"Goldman Sachs" }, { Name:"Goldman, Carl" } ]`

### Derived classes

```sql
SELECT C FROM PrivateCustomer C WHERE Name LIKE 'Goldman%'
```

Returns `[{ Name:"Goldman, Carl", Gender:"Male" }]`

```sql
SELECT C FROM CorporateCustomer C WHERE Name LIKE 'Goldman%'
```

Returns `[{ Name:"Goldman Sachs", VatNumber:"1234" } ]`

### Inheriting From Non-Database Classes

A database class cannot inherit from a class that's not a database class. This will throw, during weave-time, `System.NotSupportedException` or `ScErrSchemasDoNotMatch (SCERR15009)`depending on how the base class is defined.

It's also not possible to cast a non-database class to a database class.

## Comparing Database Objects

### Comparing in SQL Queries

Database objects can be checked for equality in SQL queries with the equals operator:

```csharp
var products = Db.SQL<Product>(
    "SELECT p FROM Product p WHERE p.Customer = ?", customer);
```

### Comparing in C\#

Database objects can be checked for equality in C\# with the `Equals`method:

```csharp
var firstProduct = new Product();
var secondProduct = new Product();
var anotherFirstProduct = 
    Db.FromId<Product>(firstProduct.GetObjectNo());

firstProduct.Equals(secondProduct); // => false
firstProduct.Equals(anotherFirstProduct); // => true
```

{% hint style="warning" %}
The equals `==` operator and the `Object.ReferenceEquals` method will always return `false`.
{% endhint %}

