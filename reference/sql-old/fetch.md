# FETCH

  
The number of returned results can be limited using the `FETCH` clause. An example can be seen below:

```sql
SELECT e.LastName, e.FirstName
  FROM Employee e
  FETCH FIRST 5 ROWS ONLY
```

The only mandatory reserved word is the word `FETCH` as in the example below; the other reserved words are optional.

```sql
SELECT e.LastName, e.FirstName
  FROM Employee e
  FETCH 5
```

The `FETCH` clause should be after the main part of the query possibly including an `ORDER BY` clause but before an `OPTION` clause including hints, see example below.

```sql
SELECT e.LastName, e.FirstName
  FROM Employee e
  ORDER BY e.FirstName
  FETCH 5
  OPTION INDEX (e MyIndexOnFirstName)
```

