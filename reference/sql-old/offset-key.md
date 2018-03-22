# OFFSET KEY

## Introduction 

Starcounter allows to retrieve query results in portions without need to keep an `Enumerator` \(a cursor\) with query result open. In addition to standard OFFSET clause, Starcounter SQL is extended with `OFFSETKEY` clause, which re-creates the query enumerator and continues after the last retrieved record.

While a server side cursor can provide a snapshot isolation, the `OFFSETKEY` functionality is a good compromise. It provides for client side cursors with no server side state, just as `OFFSET` does, but without the duplicate or missing records common with `OFFSET`.

## Creating OFFSET KEY Queries

You retrieve each portion of the query result by sending a new query, just as you do with a standard `OFFSET` query. Instead of a number indicating the position you should skip to, you instead retrieve a string value \(the offset key\) at the end of each portion. For the next portion, you provide the exact same query, but with the new key as a parameter value.

The query usually includes [FETCH](./#fetch) clause, which limits each retrieval \(each portion\). Each query will will begin to retrieve result after the last fetched row from the previous portion, taking into account any deletions, changes or insertions.

The initial query must be identical with the subsequent queries apart from the `FETCH` and `OFFSETKEY` clauses and actual fetch and offset key values supplied. The first time the query is executed \(i.e. for the first portion\), the offset key value should always be set to `null` or omitted.

The `OFFSETKEY` clause is placed at the end of the `SELECT` statement and can be together with FETCH clause, e.g.:

```sql
SELECT u FROM User u FETCH ? OFFSETKEY ?
```

The first time the query is used, you should supply the value `null` as the OFFSETKEY.

The input parameter is a string key, which is obtained on an enumerator to be re-created by calling method `GetOffsetKey()`. The string key for OFFSETKEY clause, _offset key_, can be also retrieved from the query already having OFFSETKEY clause.

{% hint style="warning" %}
You can't use `OFFSETKEY` with `ORDER BY` or `GROUP BY` clauses.
{% endhint %}

## Getting an Offset Key in Initial Query

The offset key is obtained by calling method `GetOffsetKey()` on enumerable or on enumerator, which is instance of `IRowEnumerator`. Getting offset key on enumerable can be done only if one enumerator was open.

The enumerator is obtained for initial query by calling standard interface method `GetEnumerator`, e.g.:

```csharp
IRowEnumerator<User> e = Db.SQL<User>("SELECT u FROM User u FETCH ? OFFSETKEY ?", 10, null).GetEnumerator();
```

An offset key is obtained by calling method `GetOffsetKey`, which has the following signature:

```csharp
byte[] GetOffsetKey();
```

The offset key is obtained at any valid state of enumerator, i.e., after `MoveNext` was called and was `true`:

```csharp
byte[] key = null;
using (IRowEnumerator<User> rows = Db.SQL<User>("SELECT u FROM User u").GetEnumerator())
{
    int i = 0;
    while (rows.MoveNext())
    {
        User u = rows.Current;
        ...
        i++;
        if (i == 3)
        {
          key = rows.GetOffsetKey();  
        }
    }
}
```

The offset key can be obtained after query with FETCH clause was enumerated. The offset key will be valid, if there are more rows exist after the fetched number of rows:

```csharp
byte[] key = null;
using (IRowEnumerator<User> rows = Db.SQL<User>("SELECT u FROM User u").GetEnumerator())
{
    while (rows.MoveNext())
    {
        User u = rows.Current;
        ...
    }
    key = rows.GetOffsetKey();
}
```

## Continue to Retrieve Data

To be able to recreate the `Enumerator` and continue query execution, the query with offset key, _offset key query_, should be the same as the initial query, which was used to obtain the offset key, _original query_. Both query string and query variable values should be the same. Only `FETCH` and `OFFSET KEY` clauses can be different between queries. `OFFSET` and `OFFSET KEY` cannot be used in the same query.

If an `OFFSET KEY` query is not exactly the same as the original query, except for the `FETCH` or `OFFSET KEY` clauses and  values, then Starcounter will throw an exception.

If the offset key query is the same as the original query and the offset key is not `null`, then the first row of the result of the offset key query will be the next row after the row of the original query, which was retrieved last before the offset key was gotten. The next row is defined for the moment when the offset key query is called. Thus if there were rows inserted after the last row of the original query, then the offset key query will retrieve them. Deleting the last row of the original query does not affect the result of the offset key query.

Query with `null` value for `OFFSET KEY` clause is equivalent to query with omitted `OFFSET KEY` clause.

If a new row, which has the same values as the last row of original query, \(with or without deleting the last row\) is inserted, then depending on its place in an index used in the query plan the offset key query will either start from it or after it and the last row.

## Example

```csharp
byte[] k  = null;
using (IRowEnumerator<Account> e = Db.SQL<Account>("SELECT a FROM Account a WHERE a.AccountId < ? FETCH ?", 100, 10).GetEnumerator())
{
   while (e.MoveNext()
   {
       Account a = e.Current;
       Console.Write(a.AccountId + " ");
   }
   k = e.GetOffsetKey();
}

if (k == null) return;
Console.WriteLine();

using (IRowEnumerator<Account> e = Db.SQL<Account>("SELECT a FROM Account a WHERE a.AccountId < ? FETCH ? OFFSETKEY ?", 100, 5, k).GetEnumerator())
{
   while (e.MoveNext()
   {
       Account a = e.Current;
       Console.Write(a.AccountId + " ");
   }
   k = e.GetOffsetKey();
}
...
```

If the database contains accounts with following AccountIds:

```text
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 ...
```

The code above will return:

```text
1 2 3 4 5 6 7 8 9 10
11 12 13 14 15
...
```

  
If the database contains accounts with following AccountIds:

```text
10 20 30 40 50 60 70 80 90 100 110 120 130 140 150 160 170 180 190 200 210 ...
```

The code above will return:

```text
10 20 30 40 50 60 70 80 90
```
