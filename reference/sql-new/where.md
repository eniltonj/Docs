# WHERE

The `WHERE` clause is used to extract only those records that fulfill a specified criterion.

## Syntax {#syntax}

```sql
WHERE-clause ::=
       'WHERE' predicate-expression

predicate-expression ::=
       expr infix-operator { literal | ?}

expr ::= [alias '.'] attribute-name

infix-operator ::= { '=' | '<>' | '>' | '<' | '>=' | '<='}
```

In `WHERE` clause, a predicate expression over `attribute-name` is evaluated to a boolean value. Here, `const-expression` is a constant while `attribute-name` represents an attribute of a type. Note that, `attribute-name` is referred with [identifiers](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifiers.html). For example, `Name` is an attribute of a type `Person`.

```sql
SELECT p FROM Person p 
WHERE  Name = 'Bill Gates'
```

Depending the type of`attribute-name` , the infix operator can be any of of the following comparison operators `=, <>, >, <, >=, <=` . Please refer to comparison operators for further reading which comparison operators are applicable on which type. If the `WHERE` clause in a query is otherwise than stated, the query will not be executed and `SCERRNOTIMPLEMENTED` error is returned.

Last but not least, an `alias`, which is a temporary identifier of a type, followed by `.` is used to identified that the `attribute-name` is of the type. See [identifiers](https://starcounter.gitbooks.io/rebelslounge/content/sql/Identifier.md)

```sql
SELECT p FROM Person p 
WHERE  p.Name = 'Bill Gates'
```

## Semantics {#semantics}

The `WHERE` clause in a `SELECT` statement specifies a selection filter on returned objects.

In `WHERE` clause, `?` can be used as placeholders for parameters and actual values of the parameter are supplied at execution time. The query, thus, is parameterized query. Again, in `WHERE` clause the `attribute-name` can only be an indexed attribute.

```sql
SELECT p FROM Person p WHERE p.Name = ?
```



