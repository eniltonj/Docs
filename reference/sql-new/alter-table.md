# ALTER TABLE

Alters an existing table by adding and/or dropping columns.

## Syntax

```sql
alter-table-stmt ::= 'ALTER' 'TABLE' table-name alter-table-command

alter-table-command ::= 'ADD' ['COLUMN'] column-definition
                      | 'DROP' ['COLUMN'] column-name

column-definition ::= column-name type-name [column-option]

column-option ::= NULL
                | NOT NULL
```

Here `table-name` is the name of the table to be modified. It can be provided as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces.

If `table-name` cannot be resolved unambiguously against the known set of tables, _'ambiguous type'_\(`ScErrSqlAmbiguousType`\) error occurs. If table is not found, _'type not found'_ \(`ScErrTypeNotFound`\) error occurs.

## ADD COLUMN {#add-column}

Adding a column to a table requires the same kind of `column-definition` as provided to [CREATE TABLE](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_CREATE_TABLE.html). The name of the column to add is specified as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html) `column-name`, and the type of the column can be either a [supported primitive type](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_types.md), or name of an existing table \(provided as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces.

The column is added to the table identified by `table-name`, and all the inheriting tables. If there is a conflict with an existing column, _'column name not unique'_ \(`ScErrColumnNameNotUnique`\) error occurs.

The `NULL` or `NOT NULL` option provided overrides the default _nullability_ for the column types.

### Example

```sql
ALTER TABLE Person ADD COLUMN seqno INT NULL
```

This adds an integer column `seqno` to the table `Person`. The new column is _nullable_ \(which is not the default for integer columns\).

```sql
ALTER TABLE OurCompany.Employee ADD COLUMN manager OurCompany.Employee
```

This adds a column `manager` to the table `Employee` defined in the namespace `OurCompany`. The type of the added column is the same as `Employee` table itself, i.e. this column will contain _references_ to rows in the same table. Such _reference_ columns are _nullable_ by default.

## DROP COLUMN {#drop-column}

The column to drop is specified by name `column-name`. If there is no such column in the table being modified, _'unknown column'_ \(`ScErrSqlUnknownColumn`\) error occurs.

The column is dropped from the table identified by `table-name` and all the inheriting tables. If there is an index defined in any of these tables, and this index includes the column being dropped, _'index exists on column'_ \(`ScErrIndexExistsOnColumn`\) error occurs. The table and index name are reported in the error message, so that a [DROP INDEX](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_DROP_INDEX.html) statement can be executed first.

### Example

```sql
ALTER TABLE OurCompany.Employee DROP COLUMN manager
```

drops the `manager` column which was added in the previous example.

