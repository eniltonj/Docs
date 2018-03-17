# SQL - New

This section describes the SQL syntax supported by the new query processor.

## Identifiers

In Starcounter, any `type-name`, e.g; table names, or `attribute-name` \(i.e, column name\), or name-space or so on are referred by their identifier. For example:

* The type Person is identified by identifier \(in literal \) `Person`. It has name and age as attributes of the type `Person` identified by identifiers `Name` and `Age` respectively.
* The type stores and attributes of the type such as its located street and its postal code are identified by `Stores`, `Street`, and `Postal code`.

Depending on which level \(or context\), one may have to refer to it as a list of identifier with a delimiter, which it is `.`. For example in the same database:

* A user defined a name-space `A` and under which, created two types `B` and `C`.
* Another user defined name-space `B` and under which created two more types `C` and `D`.
* Thus, to refer to the type `C` within the name-space `A`, one can use a list of identifiers as `A.C` to differentiate with the type `C` in the name-space `B` as `B.C`.

In the this example, name ambiguity is avoided by the use of list of identifiers.

### Notes {#notes}

* In Starcounter world, identifier is a single word that complies with [C\# identifier](https://msdn.microsoft.com/en-us/library/e7f8y25b.aspx).
* Syntax for list of identifiers is described as following:

```sql
list-of-identifers ::= 
    identifier
    | ['.' list-of-identifers]
```

* In additon, reserved keywords, which are words that have fixed meaning in the SQL language, can only be used as identifiers if they are escaped with `""`.
* The following words are reserved key words in Starcounter. Please bear in mid that the keyword list is subject to be changed in later version of Starcounter.

```text
    ALL ALTER ALWAYS ANALYSE ANALYZE AND ANY APPLY ARRAY AS ASC
    ASYMMETRIC AT

    BEFORE BEGIN_P BETWEEN BIGINT BIT
    BOOLEAN_P BOTH BY

    CACHE CASCADE CASCADED CASE CAST CHAR_P
    CHARACTER CHARACTERISTICS CHECK CHECKPOINT
    COALESCE COLLATE COLLATION COLUMN COMMENTS COMMIT
    COMMITTED CONCURRENTLY CONSTRAINT CONSTRAINTS
    CONTENT_P CONTINUE_P CREATE
    CROSS CSV CURRENT_P
    CURSOR CYCLE

    DATA_P DATABASE DAY_P DEALLOCATE DEC DECIMAL_P DEFAULT DEFAULTS
    DEFERRABLE DEFERRED DELETE_P DESC DESCENDANTS
    DISABLE_P DISCARD DISTINCT DO DOCUMENT_P DOMAIN_P DOUBLE_P DROP

    EACH ELSE ENABLE_P ENCODING END_P ESCAPE EXCEPT
    EXCLUDE EXCLUDING EXCLUSIVE EXECUTE EXISTS EXPLAIN
    EXTRACT

    FALSE_P FETCH FIRST_P FLOAT_P FOLLOWING FOR FORCE FOREIGN
    FROM FULL

    GLOBAL GREATEST GROUP_P

    HAVING HOLD HOUR_P

    IDENTITY_P IF_P ILIKE IMMEDIATE IN_P
    INCLUDING INCREMENT INDEX INDEXES INHERIT INHERITS INITIALLY
    INNER_P INSERT INSTEAD INT_P INTEGER
    INTERSECT INTERVAL INTO IS ISNULL ISOLATION

    JOIN

    KEY

    LAST_P LC_COLLATE_P LC_CTYPE_P LEADING
    LEAST LEFT LEVEL LIKE LIMIT LOCAL
    LOCK_P

    MAP MATCH MAXVALUE MINUTE_P MINVALUE MODE MONTH_P

    NAME_P NATIONAL NATURAL NCHAR NEXT NO
    NOT NOTNULL NOWAIT NULL_P NULLIF
    NULLS_P NUMERIC

    OBJECT_P OF OFFSET OFFSETKEY OIDS ON ONLY OPERATOR OPTION OPTIONS OR
    OUTER_P OVER OVERLAPS OVERLAY OWNED OWNER

    PARTIAL PARTITION PASSING PLACING PLANS
    PRECEDING PRECISION PRESERVE PREPARE PREPARED PRIMARY
    PROCEDURE

    RANGE READ REAL REASSIGN RECURSIVE REF REFERENCES REINDEX
    RELEASE REPEATABLE REPLACE REPLICA
    RESET RESTART RESTRICT RETURNING RIGHT ROLLBACK
    ROW ROWS

    SAVEPOINT SECOND_P SELECT SEQUENCE
    SERIALIZABLE SET SETOF SHARE
    SIMILAR SIMPLE SMALLINT SOME STANDALONE_P START STARTS 
    STATEMENT STORAGE STRIP_P SUBSTRING
    SYMMETRIC SYSTEM_P

    TABLE TABLESPACE TEMP TEMPLATE TEMPORARY THEN TIME TIMESTAMP
    TO TRAILING TRANSACTION TREAT TRIGGER TRIGGERS TRIM TRUE_P
    TRUNCATE TYPE_P TYPEOF

    UNBOUNDED UNCOMMITTED UNION UNIQUE UNLOGGED
    UPDATE USER USING

    VALID VALIDATE VALUE_P VALUES VARCHAR VARIADIC VARYING
    VERBOSE VERSION_P VIEW

    WHEN WHERE WHITESPACE_P WINDOW WITH WITHOUT WORK WRITE

    XMLATTRIBUTES XMLCONCAT XMLELEMENT XMLEXISTS XMLFOREST XMLPARSE
    XMLPI XMLROOT XMLSERIALIZE

    YEAR_P YES_P

    ZONE
```

