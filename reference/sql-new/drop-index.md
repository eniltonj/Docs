# DROP INDEX

Drops a named index.

## Syntax

```sql
drop-index-stmt ::= 'DROP' 'INDEX' index-name ['ON' table-name]
```

`index-name` is the name of the index to be dropped, and `table-name` is the name of the table, where the index should be dropped from. If the table name is omitted then the index is dropped if exists. These names can be provided as single [identifiers](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces.

If a `table-name` cannot be resolved unambiguously against the known set of tables, _'ambiguous type'_\(`ScErrSqlAmbiguousType`\) error occurs. If table is not found, _'type not found'_ \(`ScErrTypeNotFound`\) error occurs.

If a named index cannot be found, _'index not found'_ \(`ScErrIndexNotFfound`\) error occurs. If `table-name`is specified, the additional check is performed to verity that the index is defined on the given table.

### Example

The following statement drops index `Person_Index` from table `Person`. If the index doesn't exist in the table, an error will be returned:

```sql
DROP INDEX Person_Index ON Person
```

The following statement drops index `Person_Index` if it exists in any table:

```sql
DROP INDEX Person_Index
```

