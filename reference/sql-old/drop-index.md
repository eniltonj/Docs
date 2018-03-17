# DROP INDEX

  
Existing indexes can be dropped from a database by query with syntax `DROP INDEX index_name ON table_name`. For example:

```csharp
Db.SQL("DROP INDEX EmpDepartmentSalaryIndex ON Employee");
```

