# CREATE INDEX

Indexes are declared in an application using method `Db.SQL(String query)`, where query string contains an index declaration and has the form `CREATE [UNIQUE] INDEX indexName ON typeName (propertyName [ASC/DESC], ...)`.

It is recommended to declare indexes before any retrieval query is issued. **Note** that indexes should be declared **outside** a transaction scope.

In the examples \(1\), \(2\), \(3\), \(4\) and \(5\) below we declare indexes on different properties/columns of the class/table Employee.

```sql
(1) CREATE INDEX EmpFirstNameIndex ON Employee (FirstName ASC)

(2) CREATE INDEX EmpLastNameIndex ON Employee (LastName ASC)

(3) CREATE INDEX EmpSalaryIndex ON Employee (Salary)

(4) CREATE INDEX EmpManagerIndex ON Employee (Manager)

(5) CREATE INDEX EmpDepartmentIndex ON Employee (Department)
```

The default order, used when there is no order declared, is ASC \(ascending\). The order specified in the index declaration only matters when you want the result set to be sorted \(ORDER BY\).

You can declare indexes on properties/columns of the following datatypes \(DbTypeCode\): Boolean, Byte, DateTime, Decimal, Int16, Int32, Int64, Object, SByte, String, UInt16, UInt32, UInt64.

You can declare combined indexes on up to ten different properties/columns of a class/table. In the examples \(6\), \(7\), \(8\) and \(9\) we have some combined indexes on two properties/columns of the class/table Employee.

```sql
(6) CREATE INDEX EmpLastnameFirstNameIndex ON Employee (LastName ASC, FirstName ASC)

(7) CREATE INDEX EmpFirstNameLastNameIndex1 ON Employee (FirstName ASC, LastName ASC)

(8) CREATE INDEX EmpFirstNameLastNameIndex2 ON Employee (FirstName DESC, LastName ASC)

(9) CREATE INDEX EmpDepartmentSalaryIndex ON Employee (Department ASC, Salary DESC)
```

## Checking for declared indexes

The `indexName` must be unique. If you define the same name more than once you will get an exception. It is possible to check if an index was already created by issuing a query, which selects a record from table `Starcounter.Metadata.Index` with column `Name` equivalent to the index name as in the example below.

```csharp
if (Db.SQL("SELECT i FROM Starcounter.Metadata.\"Index\" i WHERE Name = ?", "EmpDepartmentSalaryIndex").FirstOrDefault() == null)
    Db.SQL("CREATE INDEX EmpDepartmentSalaryIndex ON Employee (Department ASC, Salary DESC)");
```

## Derived indexes

The current version do not support derived indexes. You need to define index on the class you like to query. For instance, say we have the following structure:

```csharp
[Database]
public class LegalEntity
{
   public string Name { get; set; }
}

public class Company : LegalEntity
{}

public class Person : LegalEntity
{}
```

You will need to define index on Name for both Company and Person.

If index is defined on a database property for the base class, the query optimizer might choose to use it in queries on a child class of the base class, but this will require to filter out all instances, which are not of the child class. For example, index is created on `Name` only for `LegalEntity` and a query is submitted for `Company`, then if the query optimizer chooses to use the index, it will add a filter predicate, which checks that all objects from the index are instances of `Company`.

