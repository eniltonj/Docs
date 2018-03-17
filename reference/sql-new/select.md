# SELECT

The `SELECT` statement is used to select data from a database. The result of the query has no specific order.

## Syntax {#SELECT_syntax}

```sql
select-stmt ::=
        'SELECT' SELECT-clause
                 FROM-clause
                 [WHERE-clause]

SELECT-clause ::= alias
FROM-clause   ::= type-name [AS] alias
```

In which, `type-name` is an identifier or list of identifiers of a type while **`alias`** can used to give `type-name` a temporary identifier in query context.

For example, user-defined types `A` and `A.B` are stored in the database with full path as `A` and `A.B`. Thus, the following SELECT queries are all valid:

```sql
SELECT a FROM A a
SELECT b FROM A.B b
SELECT b FROM B b
```

The type `B` can be addressed by its full name as in `line 2` or short name as in `line 3`. However, addressing a type with its short name may potentially cause [name ambiguity](select.md#descriptions-of-errors).

## Purpose {#purpose}

The `SELECT` statement selects all type instances belonging to the `type-name` if there is no `WHERE` clause specified.

The `WHERE` clause is optional but when given, it indicates that any type instances that evaluates the `where_condition` to true will be selected. See [`WHERE`](where.md) clause for the full details.

On success, an iterator, which allows iterate over the selected instances, is returned. In addition, a `SCSQL_ERROR` containg `error code = 0` and an empty `error message` is returned. On failure, instead, a specific error code, together with and meaningful error message describe the cause of error. Note that, the returned iterator of course is invalid to use. Iterating of result does not guarantee any order if no `ORDER BY` specified.

The following subsections lists all cases resulting different type of errors that may occur when executing the [SELECT statement](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_SELECT.html).

## Exceptions {#Descriptons-of-errors}

### Incorrect Syntax {#incorrect-syntax}

SQL queries that are syntactically incorrect will throw `SCERRSQLINCORRECTSYNTAX`.

### Not Implemented {#not-yet-implemented-error}

The input [SELECT statement](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_SELECT.html) must strictly follow the supported SQL syntax described above. If the statement uses clauses outside of what's supported, the query processor throws `SCERRNOTIMPLEMENTED`.

### Type not Found {#type-not-found}

If there is no type matching _`type-name`_, the `SELECT` statement is not executed and `SCERRTYPENOTFOUND` is thrown.

### Name Ambiguity {#name-ambiguity}

When there are more than one type in the database matching _`type-name`_, the name is ambiguous and `SCERRSQLAMBIGUOUSTYPE`is thrown. If there are types named `Aa.B`, and `A.B`in additional to the previous existing type `A`, the following query cause ambiguous error:

```sql
SELECT b FROM B b
```

It is because the given `B` can be match with existing types `Aa.B` or `A.B`

## Choosing Index {#The-choice-of-index}

Selecting data from a type does not requires the presence of any index on the type.

If there is an index on the queried type, that index will be used in the execution of the `SELECT` statement to retrieve type instances. When such index is missing, the execution of `SELECT` statement uses the global index `MotherOfAllLayout_SetSpec_Index` instead.

There is case that, a type does not have any index but its bases do. Thus, any inherited index can be used together with a filtering out irrelevant types. However, we have proof of concept of performance, hence, the use of the global index is favored over inherited indexes.

## Result Order {#Order-of-retrieved-result}

Depending on how entries are stored in the index used to retrieve type instances, the order of the result may vary.

If an ascending index is used, the result set is in ascending order. On other hand, when a descending index is used, the result set is in descending order.

When the global index is used because of lacking of index on the queried type, the order of the result set is undefined.

