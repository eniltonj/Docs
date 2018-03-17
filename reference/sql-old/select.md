# SELECT

## Introduction

`SELECT` queries is the way to fetch data with SQL. Starcounter supports `SELECT` queries with [aggregates,](select.md#aggregates) [data operators](select.md#data-operators), and [casting](select.md#casting).

It has this syntax:

```sql
select-stmt ::=
        'SELECT' SELECT-clause
                 FROM-clause
                 [WHERE-clause]

SELECT-clause ::= alias
FROM-clause   ::= type-name [AS] alias
```

The result of `SELECT` has no particular order. To order the result, use [`ORDER BY`](order-by.md).

## Aggregates

Starcounter supports aggregates with grouping \(`GROUP BY`\), conditions on groups \(`HAVING`\), and the standard set functions `AVG`, `SUM`, `COUNT`, `MAX` and `MIN`.

```sql
SELECT AVG(e.Salary), MAX(e.Salary), MIN(e.Salary), e.Department
  FROM Example.Employee e
  GROUP BY e.Department
  HAVING SUM(e.Salary) > 20000
```

### Using Asterisk Shorthand With `COUNT`

The asterisk shorthand is treated as a literal in `COUNT`. Since `Db.SQL` doesn't support literals, using `COUNT(*)` in `Db.SQL` will throw `ScErrUnsupportLiteral`:

```text
ScErrUnsupportLiteral (SCERR7029): Literals are not supported in the query. Method Starcounter.Db.SQL does not support queries with literals. Found literal is 1. Use variable and parameter instead.
```

 There are three ways to work around this:

```csharp
// This throws SCERR7029
Db.SQL("SELECT COUNT(*) FROM Person").First();
// Using an identifier instead of * works
Db.SQL("SELECT COUNT(p) FROM Person p").First();
// Db.SlowSQL supports literals, so it can be used
Db.SlowSQL("SELECT COUNT(*) FROM Person").First();
// Linq can also give you the number of rows
Db.SQL("SELECT p FROM Person p").Count();
```

The first option of using an identifier to get the count best in most cases, both for versatility and performance.

## Data Operators

In Starcounter SQL only the most common operators on data are implemented.

The standard arithmetic operators, plus \(+ x\), minus \(- x\), addition \(x + y\), subtraction \(x - y\), multiplication \(x \* y\) and division \(x / y\), are supported for all numerical types. See example below.

```sql
SELECT (e.Salary * 12) / 365 FROM Employee e
```

String concatenation \(x \|\| y\) is supported. See example below.

```sql
SELECT e.FirstName || ' ' || e.LastName FROM Employee e
```

For the expected datatypes of an arithmetic operation, see [Datatypes](../../topic-guides/database/data-types.md).

## Casting

In some path expressions you need to cast the type of a property/column.

Assume `Employee `is a sub-type of `Person`, the property `Father` is of type `Person` and is defined in `Person`, the property `Manager` is of type `Employee` and is defined in `Employee`, and you want to select the manager of each person's father whenever such manager exists.

```csharp
[Database]
public class Person
{
  public Person Father { get; set; }
}

[Database]
public class Employee : Person
{
  public Employee Manager { get; set; }
}
```

In this case the query below would be incorrect because `Father` is of type `Person` and this type have no property called `Manager`.

```sql
/* incorrect */
SELECT p.Father.Manager FROM Person p
```

However, if you cast `Father` to type `Employee` then you can continue the path expression with `Manager`. See example below.

```sql
/* correct */
SELECT CAST(p.Father AS Employee).Manager FROM Person p
```

If the object reference `Father` for some objects in the extent `Person` is not of type \(or a sub-type of\) `Employee` then this object reference cannot be cast to `Employee`. In such cases the cast operation returns null and consequently the complete path expression also returns `null`.

Currently, the cast operation only supports casts between different types of database objects, which are of type `Entity`. The cast operation does not support type conversion between different value types.

