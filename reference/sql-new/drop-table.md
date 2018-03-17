# DROP TABLE

Drops existing tables.

## Syntax

```sql
drop-table-stmt ::= 'DROP' 'TABLE' table-name [',' table-name]* ['CASCADE' 'DESCENDANTS']
```

Here `table-name` is the name of the table\(s\) to be dropped. These names can be provided as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces.

If a `table-name` cannot be resolved unambiguously against the known set of tables, _'ambiguous type'_\(`ScErrSqlAmbiguousType`\) error occurs. If table is not found, _'type not found'_ \(`ScErrTypeNotFound`\) error occurs.

## CASCADE DESCENDANTS {#cascade-descendants}

Tables can be created inheriting from other tables, e.g. by providing _base table name_ in [CREATE TABLE](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_CREATE_TABLE.html)statement. By default, a table that has descendants cannot be dropped - a _'table has children'_\(`ScErrTableHasChildren`\) error occurs.

This can be altered by `CASCADE DESCENDANTS` option: if specified, all tables directly or transitively inheriting from the ones listed in `DROP TABLE` statement, will be dropped as well.

### Example

Assume the following tables were created:

```sql
CREATE TABLE Person (name VARCHAR)
CREATE TABLE Employee (salary NUMERIC) INHERITS Person
CREATE TABLE Consultant (partner Company) INHERITS Employee
```

The`Person` table has a direct child table `Employee`, and the latter has a direct child table `Consultant`. Thus `Employee` and `Consultant` are said to be the _descendants_ of `Person`. Dropping tables `Person` or `Employee` without `Consultant` table is not possible by default.

```sql
DROP TABLE Person CASCADE DESCENDANTS
```

This will drop`Employee` and `Consultant` without mentioning them in the `DROP TABLE` statement - along with any other descendants of `Person`.

## Column Types as References {#column-types-as-references}

Tables can reference each other as column types. If a table being dropped is referenced as a column type by another table, and that other table is not listed in DROP TABLE statement - the statement will fail with '_table is referenced_' `ScErrTableIsReferenced (scerr15012)` error.

Instead of dropping or altering the other table first, both tables can be listed in the same statement. The order is not important, as tables can mutually refer to each other.

### Example

The previous example assumes there is a table `Company` which serves as a type for the column `partner` in table `Consultant`. Trying to drop table `Company` will not work, since the table `Consultant`will become impossible to maintain. However, if `Consultant` table is also meant to be dropped, this query can be used:

```sql
DROP TABLE Company, Consultant
```

