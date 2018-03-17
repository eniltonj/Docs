# SQL - Old

Starcounter SQL follows the SQL-92 standard to support easy data exchange with other databases and external tools.

## Calling SQL

SQL is called in code with `Db.SQL` as described in [Querying Using SQL](../../topic-guides/database/querying-using-sql.md).

In this reference, SQL queries are written out literals for readability. To use these queries in code, use [Variables](../../topic-guides/database/querying-using-sql.md#variables) instead of literals.

## Object extensions to SQL

Starcounter SQL contains some extensions to the SQL92 standard to better deal with objects, since the standard SQL only supports relational databases. For these extensions we follow the Object Data Standard ODMG 3.0 \(ISBN 1-55860-647-4\). The object extensions in Starcounter SQL are:

* object references,
* [path expressions]().

In traditional SQL you can only refer to tables, columns, rows and fields of values. The concept of an "object" is represented by a row in a table. You refer to an "object" by values on its primary key which are some specified columns.

In Starcounter SQL you can refer to an object itself. For example, in query below the identifier `e` is an object reference.

```sql
SELECT e FROM Employee e
```

In a traditional SQL database, to get data from more than one type of object \(table/class\) you have to do a "join" of a number of tables. For example, query below gives you the names of the employees and the names of the departments where they work.

```sql
SELECT e.FirstName, d.Name FROM Employee e JOIN Department d ON e.DepartmentId = d.Id
```

In Starcounter SQL there is a more convenient way to get the same result by instead using a path expression as in query below. In that way, you can from one type of object \(`Employee`\) reach another type of object \(`Department`\) by object reference.

```sql
SELECT e.FirstName, e.Department.Name FROM Employee e
```

In object oriented programming the extent of a class is all object instances of that class. An extent of object instances corresponds to a table of rows in a relational database. Thus, `Employee` in the example queries above can either be regarded as the extent of the class `Employee` or as the table `Employee`.

## Identifiers

An identifier \(class/table name, property/column name etc.\) in Starcounter SQL is a sequence of the following characters:

* lower case letters \[ a - z \],
* upper case letters \[ A - Z \],
* digits \[ 0 - 9 \] or
* underscore \[ \_ \],

where the first character may not be a digit \[ 0 - 9 \].

Note that Starcounter SQL is case insensitive.  This means that identifiers which are equal except for different cases of the letters are regarded to be the same identifier. Consequently you cannot have different identifiers in your database schema with the same name but different cases.

Starcounter automatically creates a database schema from the class definitions in the application program code. Thus you could have several classes/tables with the same name but in different namespaces. As a consequence, in Starcounter SQL you qualify a class/table name by specifying its namespace, not by specifying a database name or schema name. See example query below.

```sql
SELECT m.MyProperty FROM MyCompany.MyApplication.MyModule.MyClass m
```

As long as the class/table name is unique you do not have to specify the namespace. See example query below. However, in SQL statements in programming code we strongly recommend you to qualify all class/table names so the SQL statements will be guaranteed to still be valid when you add new classes/tables.

```sql
SELECT m.MyProperty FROM MyClass m
```

If you have some identifier in your database schema that conflicts with some  
[reserved words](./#reserved-words) in Starcounter SQL, you can tell the SQL parser that the term should not be interpreted as the reserved word by putting it inside double quotes, as in query below.

```sql
SELECT n."Left", n."Right" FROM Node n
```

## Path Expressions

In standard SQL92 you can only refer to columns \(properties\) of the tables \(extents\) which are specified in the FROM clause of the SELECT statement. In contrast to that, in Starcounter SQL you can refer to any property/column of an object in any extent/table as long as there is a path from an extent/table specified in the FROM clause to that property/column.

A path expression is an arbitrary long sequence of identifiers separated by points. The first identifier in the sequence should be an alias to an extent/table defined in the FROM clause of the SELECT statement. The following identifiers  except the last one should have object references as return values. The last identifier may return any supported datatype. See example below.

```sql
SELECT e.Manager.Department.Location.Street FROM Employee e
```

Note that if some identifier in a path expression will return null then the complete path expression will also return null.

In fact, you do not have to qualify \(start with an extent/table alias\) a path expression. You are allowed to omit the qualifications of the path expressions \(column references\) as in the   query \(2\) below.

```sql
SELECT Manager.Department.Location.Street, Name FROM Employee JOIN Department ON DepartmentId = Id
```

However, you are only allowed to omit the qualifications as long as the names of the  properties/columns are unique, and this may change over time. Therefore we strongly recommend that when you write SQL statements in program code you always qualify all path expressions so your code will hold for future modifications of the database schema.

You are also allowed to use a wildcard \(\*\) to select all properties/columns of an extent/table as in the queries below.

```sql
SELECT * FROM Employee
SELECT e.* FROM Employee e
```

Note that the above queries return all properties/columns of the Employee objects, while the below query returns references to the Employee objects themselves.

```sql
SELECT e FROM Employee e
```

## Index Recommendations

For all SELECT statements in your programming code, it is recommended to declare, when possible, indexes for:

* all conditions in WHERE clauses,
* all join conditions,
* all sort specifications in ORDER BY clauses.

An execution of the query \(11\) below can make use of an index on the property/column FirstName such as the index \(1\) above. It can also make use of combined indexes such as \(7\) or \(8\) where the first property/column of the index is FirstName. It can not make use of the combined index \(6\) where FirstName is not the first property/column of the index.

An execution of the query \(12\), where we have two equality comparisons on FirstName and LastName, can make use of any of the combined indexes \(6\), \(7\) and \(8\), where the two first properties/columns of the indexes are FirstName and LastName.

An execution of the query \(13\) can efficiently make use of the combined indexes \(7\) and \(8\) because the first property/column of the index is FirstName, on which we have an equality comparison, and the second property/column of the index is LastName, on which we have a range comparison. However, an execution of this query can not efficiently make use of the combined index \(6\) because the range condition on the first property/column LastName of the index makes it not possible to use the equality condition on the subsequent property/column FirstName.

An execution of the join in the query \(14\) can make use of the indexes \(5\) or \(9\), since we need an index to efficiently find all Employee objects/rows for a particular Department object/row.

An execution of the query \(15\) can make use of the combined index \(6\) to find all Employee objects/rows in the requested order without any sorting.

An execution of the query \(16\) can also make use of the combined index \(6\) to find all Employee objects/rows in the requested order without any sorting, since the index can be traversed in the reverse order.

An execution of the query \(17\) can make use of the combined index \(7\) in the reverse order to find all Employees objects/rows in the requested order without any sorting order. However, an execution of this query can not efficienlty make use of the combined index \(8\) since it neither in the normal nor the reverse order match the requested order.

```sql
(11) SELECT e FROM Employee e WHERE e.FirstName = ?

(12) SELECT e FROM Employee e WHERE e.FirstName = ? AND e.LastName = ?

(13) SELECT e FROM Employee e WHERE e.FirstName = ? AND e.LastName > ?

(14) SELECT e, d FROM Employee e JOIN Department d
         ON e.Department = d WHERE d.Name = ?

(15) SELECT e FROM Employee e ORDER BY e.LastName ASC, e.FirstName ASC

(16) SELECT e FROM Employee e ORDER BY e.LastName DESC, e.FirstName DESC

(17) SELECT e FROM Employee e ORDER BY e.FirstName DESC, e.LastName DESC
```

## Literals

For performance reasons, it is strongly discouraged to use literals in code. Instead, use [variables](../../topic-guides/database/querying-using-sql.md#variables).

A boolean literal can have one of the two values true and false, which are represented by the two reserved words `TRUE` and `FALSE`. See example below.

```sql
SELECT e FROM Employee e WHERE e.Commission = TRUE
```

There are three types of numerical literals `Int64`, `Decimal` and `Double`.

An `Int64` literal is described by its integer value, as in example below.

```sql
SELECT e FROM Employee e WHERE e.Salary = 5000
```

A `Decimal` literal is described by its numerical value including a decimal point, as in example below.

```sql
SELECT e FROM Employee e WHERE e.Salary = 5000.00
```

A `Double` literal is described by two numerical values, the mantissa and the exponent, separated by the character `E`. The mantissa may include a decimal point, but the exponent may not. See example below.

```sql
SELECT e FROM Employee e WHERE e.Salary = 5.0E3
```

A string literal is a sequence of characters beginning and ending with single quote characters. To represent a single quote character within a String literal, you write two consecutive single quote characters, as in example below.

```sql
SELECT p FROM Photo p WHERE p.Description = 'Smith''s family'
```

A date-time literal is either described by the reserved word `DATE` followed by a `String` literal of the form `yyyy-mm-dd`, the reserved word `TIME` followed by a `String` literal of the form `hh:mm:ss[.nnn]` \(the specification of milliseconds is optional\), or the reserved word `TIMESTAMP` followed by a `String` literal of the form `yyyy-mm-dd hh:mm:ss[.nnn]`. See examples below.

```sql
SELECT e FROM Employee e WHERE e.HireDate = DATE '2006-11-01'
SELECT e FROM Employee e WHERE e.HireDate = TIMESTAMP '2006-11-01 00:00:00'
```

Note that all date-time literals in fact are timestamps, which means that the date-time literal in the first query above does not represent the date `'2006-11-01'` but in fact the first millisecond of that date `'2006-11-01 00:00:00.000'`. Consequently, above examples are equivalent.

A binary literal is described by the reserved word `BINARY` and the binary value represented by a Hexadecimal string, as in example below.

```sql
SELECT d FROM Department d WHERE d.BinaryId = BINARY 'D91FA24E19FB065A'
```

Since Starcounter SQL supports object references, you also need a way to represent an object reference to a specific object, i.e. an object literal. Every object in a Starcounter database can be identified by its unique object-id-number. You describe an object literal by the reserved word `OBJECT` followed by the object's object-id-number, as in example below.

```sql
SELECT e FROM Employee e WHERE e = OBJECT 123
```

## Query Plan Hints

The Starcounter SQL optimizer decides the execution plan of an SQL-query. If you want to hint the optimizer that you prefer some particular join order or that you prefer some particular indexes to be used, you can do that in the OPTION clause at the end of the SQL statement.

To specify a preferred join order to use, you write JOIN ORDER \(extent-alias-sequence\) in the OPTION clause. You do not need to specify the order of all included extents in the extent-alias-sequence, only the ones for which you have a preferred join order. See example below.

```sql
SELECT d.Name, e.LastName FROM Department d 
  JOIN Employee e ON e.Department = d 
  WHERE e.FirstName = 'Bob' 
  OPTION JOIN ORDER (e,d)
```

If it is not possible to execute a query in the join order specified in the OPTION clause, which can be the case for outer joins, then the optimizer choses the join order to use. If you specify several join order hints only the first one will be considered.

To specify a preferred index to use for a particular extent/table, you write  INDEX \(extent-alias index-name\) in the OPTION clause. If some specified index does not exist then the optimizer choses another index if there is one. See example below.

```sql
SELECT e.FirstName, e.LastName FROM Employee e 
  WHERE e.FirstName = 'John' AND e.LastName = 'Smith' 
  OPTION INDEX (e MyIndexOnLastName)
```

You can specify an index to use for each extent in the SQL-query as in example  below. If you specify more than one index hint for a particular extent only the first one will be considered.

```sql
SELECT e, m FROM Employee e JOIN Employee m ON e.Manager = m 
  WHERE e.FirstName = 'John' AND e.LastName = 'Smith'
  AND m.FirstName = 'David' AND m.LastName = 'King' 
  OPTION INDEX (e MyIndexOnLastName), INDEX(m MyIndexOnFirstName)
```

You can specify both one index hint for each extent and one join order hint in the OPTION clause of a query, which is exemplified in example below.

```sql
SELECT e, m FROM Employee e JOIN Employee m ON e.Manager = m 
  WHERE e.FirstName = 'John' AND e.LastName = 'Smith' 
  AND m.FirstName = 'David' AND m.LastName = 'King' 
  OPTION JOIN ORDER (e, m), INDEX (e MyIndexOnLastName), 
  INDEX (m MyIndexOnFirstName)
```

## Reserved Words

The following words are reserved words in Starcounter SQL.

`ALL`, `AND`, `AS`, `ASC`, `AVG`,  
`BY`, `BINARY`,  
`CAST`, `COUNT`, `CREATE`, `CROSS`,  
`DATE`, `DATETIME`, `DELETE`, `DESC`, `DISTINCT`,  
`ESCAPE`, `EXISTS`,  
`FALSE`, `FETCH`, `FIRST`, `FIXED`, `FORALL`, `FROM`, `FULL`,  
`GROUP`,  
`HAVING`,  
`IN`, `INDEX`, `INNER`, `INSERT`, `IS`,  
`JOIN`,  
`LEFT`, `LIKE`, `LIMIT`,  
`MAX`, `MIN`,  
`NOT`, `NULL`,  
`OBJ`, `OBJECT`, `OFFSET`, `OFFSETKEY`, `ON`, `ONLY`,  
`OPTION`, `OR`, `ORDER`, `OUT`, `OUTER`, `OUTPUT`,  
`PROC`, `PROCEDURE`,  
`RANDOM`, `RIGHT`, `ROWS`,  
`SELECT`, `STARTS`, `SUM`,  
`TIME`, `TIMESTAMP`, `TRUE`,  
`UNIQUE`, `UNKNOWN`, `UPDATE`,  
`VALUES`, `VAR`, `VARIABLE`,  
`WHEN`, `WHERE`, `WITH`.

Be aware that the list of reserved words might be extended in later versions of Starcounter SQL. In particular some keywords in SQL92 might become reserved words in Starcounter SQL.

Reserved words cannot be used in queries directly. They have to be surrounded with double quotes as in example:

```sql
SELECT d FROM "DATE" d
SELECT o FROM "ORDER" o
```

Double quoting can be applied to any identifier, but only necessary for reserved keywords. It is important to double quote each identifier in identifier change, e.g.:

```sql
SELECT t FROM "Order"."Date" t
```

## SQL Application Isolation

Applications running in the same code-host are isolated on different levels:

* REST handlers and URIs namespace.
* SQL classes and objects.
* Static file resources.

The principle for SQL isolation is that the database classes of one application should not be visible to the database classes of another application running in the same code-host.

### Isolation Example

For example, the first application defines a database class `App1Class` in its namespace:

```csharp
namespace App1
{
    [Database]
    public class App1Class
    {
        public string App1ClassField { get; set; }
    }
}
```

In the second application, the class `App2Class` is defined:

```csharp
namespace App2
{
    [Database]
    public class App2Class
    {
        public string App2ClassField { get; set; }
    }
}
```

The first application is now able to access its own `App1Class` using full and short names:

```csharp
var result = Db.SQL("SELECT c FROM App1.App1Class c").FirstOrDefault();
result = Db.SQL("SELECT c FROM App1Class c").FirstOrDefault();
```

The same appplies to the second application with the class `App2Class`.

However, the first application will not be able to retrieve classes from the second application and vice versa. For example, the following code will throw an exception `Unknown class App2Class`:

```csharp
var result = Db.SQL("SELECT c FROM App2Class c").FirstOrDefault();
```

Classes defined in private application references, such as private libraries, are only accessible within the application that references them.

### Shared Library

If the first and second application are referencing the same library, for example "SharedDll", then both applications have access to classes and objects from this shared library, no matter which application created those objects:

```csharp
namespace SharedDll
{
    [Database]
    public class SharedDllClass
    {
        public string SharedDllClassField { get; set; }
    }
}
```

With this, both applications are able to query the `SharedDllClass`:

```csharp
var x = Db.SQL("SELECT c FROM SharedDllClass c").FirstOrDefault();
var x2 = Db.SQL("SELECT c FROM SharedDll.SharedDllClass c").FirstOrDefault();
```

The usa of shared libraries is a way for several applications to share the same class definitions. If you have several applications that are required to use the same classes, you will need to create a shared library and move all common class definitions there. In rare cases whenever this is not possible and you still need to have several applications accessing each other classes, you can reference other applications from you "main" application, so only one application is started.

### SQL Queries in the Administrator

Currently, the Starcounter Administrator only supports SQL queries with fully namespaced class names. In the above example, only the following queries are legitimate:

```sql
SELECT c FROM App1.App1Class c
SELECT c FROM App2.App2Class c
SELECT c FROM SharedDll.SharedDllClass c
```

## Query for Database Tables

To get the tables in the database, use the class `ClrClass`.

For example, to get all tables in a database, use this query:

```sql
SELECT * FROM ClrClass
```

This would give back all the tables, including the built-in ones.

To find all user-created tables, it is possible to use the following query:

```sql
SELECT * FROM ClrClass c WHERE c.Updatable=true AND c.UniqueIdentifier NOT LIKE 'Simplified%' AND c.UniqueIdentifier NOT LIKE 'Concepts.Ring%'AND c.UniqueIdentifier NOT LIKE 'Starcounter.%' AND c.UniqueIdentifier NOT LIKE 'SocietyObjects%'
```

