# CREATE TABLE

  
Creates a new table with a specified set of columns.

## Syntax

```sql
create-table-stmt ::= 'CREATE' 'TABLE' table-name '(' column-definition [',' column-definition]* ')' 
                          ['INHERITS' base-table-name]

column-definition ::= column-name type-name [column-option]

column-option ::= NULL
                | NOT NULL
```

Here `table-name` is the name of the table to be created. Should be fully qualified by namespaces, if applicable.

If table specified by `table-name` already exists, and the definition of that table matches the one provided in this CREATE TABLE statement, the statement completes with no effect. If definitions do not match _'schemas do not match'_ \(`ScErrSchemasDoNotMatch`\) error occurs.

## Column Definitions {#column-definitions}

For each column in the table to be created, a `column-definition` is supplied.

The name of the column to add is specified as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html) `column-name`, and the type of the column can be either a [supported primitive type](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_types.md), or name of an existing table \(provided as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces. The `NULL` or `NOT NULL` option provided overrides the default _nullability_ for the column types.

The column is added to the table identified by `table-name` and all the inheriting tables. If there is a conflict with an existing column, _'column name not unique'_ \(`ScErrColumnNameNotUnique`\) error occurs.

### Example

```sql
CREATE TABLE Person (ssn INT, name VARCHAR NOT NULL)
```

This creates a table `Person` with column `ssn` of  type integer, and column `name` of  type string , which is _not nullable_ \(string-typed columns are _nullable_ by default\).

## Inheritance {#inheritance}

The table being created can inherit its structure from another table, specified by `base-table-name` - a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces.

If `base-table-name` cannot be resolved unambiguously against the known set of tables, _'ambiguous type'  \(_`ScErrSqlAmbiguousType`\) error occurs. If the base table is not found, _'type not found'  _\(`ScErrTypeNotFound`\) error occurs.

### Example

```sql
CREATE TABLE Employee (salary NUMERIC) INHERITS Person
```

This Creates a table `Employee`, inheriting from table `Person`, having all columns defined for `Person`, plus an extra column `salary`, which is financial-precision decimal.

