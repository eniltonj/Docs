# CREATE INDEX

Creates an index on one or several columns in a table.

## Syntax

```sql
create-index-stmt ::= 'CREATE' 'INDEX' index-name 
                          'ON' table-name '(' column-index-spec [',' column-index-spec]* ')'

column-index-spec ::= column-name ['ASC' | 'DESC']
```

Here `index-name` is the name of the index to be created. It can be qualified by namespaces. If an index with the same name already exists, an _'index already exists'_ \(`ScErrNamedIndexAlreadyExists`\) error occurs.

A `table-name` is the name of the table to be indexed. It can be provided as a single [identifier](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html), or fully \(or partially\) qualified by namespaces.

If a `table-name` cannot be resolved unambiguously against the known set of tables, _'ambiguous type'_\(`ScErrSqlAmbiguousType`\) error occurs. If table is not found, _'type not found'_ \(`ScErrTypeNotFound`\) error occurs.

Column\(s\) to build an index upon are specified by `column-name` as plain [identifiers](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html). If multiple columns are listed, the order of columns is important.

For each column, an optional `ASC` \(_ascending_\) or `DESC` \(_descending_\) order specifier can be provided. For all types of columns, _ascending_ is the default order for indexing.

### Example

```sql
CREATE INDEX Person_Index ON Person (name, birth_date DESC)
```

This will create a _compound index_ on table `Person` combining the values of its `name` and `birth_date`columns in the given order. For each group of people with the same names, the rows will be sorted by `birth_date` from youngest to oldest.

