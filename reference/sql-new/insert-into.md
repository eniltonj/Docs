# INSERT INTO

  
The [INSERT INTO statement](https://starcounter.gitbooks.io/rebelslounge/content/sql/sql_INSERT_INTO.html) is used to insert data into an existing type.

## Syntax {#INSERT_syntax}

```sql
insert-stmt ::=
   'INSERT' 'INTO' type-name attribute-list
                   'VALUES'  value-list 

attribute-list ::=  (attribute-name [, attribute-name ]...)
value-list     ::= (literal [, literal ]...)
```

Here, `type-name` is an identifier or list of identifier of a type.

The following is an example of the statement if one inserts some instances into type `T` having two attributes, namely, `A` of type `STAR_TYPE_LONG`and `B` of type `START_TYPE_STRING` as the following:

```sql
INSERT INTO T(A, B) VALUES (1, 'This is a dog'),
                           (2, 'This is a cat')
                           (3, 'This is a bird')
```

## Semantics {#Semantics}

In the statement, an identifier for each attribute in the `attribute-list` of the given `type-name` must be present.

Recall the example table above, the following `INSERT INTO` statements are failed to execute and they result in incorrect syntax [error](insert-into.md#incorrect-syntax).

```sql
/* ERROR: Missing list of attributes */
INSERT INTO T VALUES (1, 'This is a dog'),
                     (2, 'This is a cat')
                     (3, 'This is a bird');
/* ERROR: Missing attribute B */                     
INSERT INTO T(A) VALUES (1, 'This is a dog'),
                     (2, 'This is a cat')
                     (3, 'This is a bird');
```

For every specified `attribute-name`, there must be a literal accordingly. It is a "one-to-one" mapping. The order in `attribute-list` and `value-list` should be in sync. If such order is not respected, the `INSERT INTO` statement results in failure when executed.

For example, consider the some valid and invalid `INSERT INTO` statements:

```sql
/*Valid statements*/
INSERT INTO T(A, B) VALUES (1, 'This is a dog'),
                           (2, 'This is a cat')
                           (3, 'This is a bird');

INSERT INTO T(B, A) VALUES ('This is a dog', 1),
                           ('This is a cat', 2)
                           ('This is a bird', 3)

/* ERROR: Missing corresponding literals */
INSERT INTO T(A,B) VALUES ('This is a dog');
INSERT INTO T(A,B) VALUES (2);
```

## Exceptions {#Descriptions-of-errors}

### Incorrect Syntax {#incorrect-syntax}

SQL queries that are syntactically incorrect will throw `SCERRSQLINCORRECTSYNTAX`.

### Constraint Violation {#constraint-violation}

If an `INSERT INTO` statement causes violation of unique constraint on some column, the statement is not executed and an error code `SCERRINVALIDSTATEMENT` is returned.

Example: Assume that column A is unique. Thus the first statement is successful while the second results in `SCERRINVALIDSTATEMENT`error.

```sql
INSERT INTO T(A, B) VALUES (1, 'This is a dog');
```

and

```sql
INSERT INTO T(A, B) VALUES (1, 'This is a cat');
```



