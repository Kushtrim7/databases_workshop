# SQL Part 1

## Connecting

To connect to a database, you need the following information:

* Host/server name: 
* Database name: dvdrental
* Username: 
* Password: 
* Port: 5432 (default for PostgreSQL)

You also need a client program to connect to the database.  It's suggested that you start with `psql`, which is a command-line client.  Output below is from `psql`.


# Database Schema

We're working with the `dvdrental` database from this [PostgreSQL Tutorial](http://www.postgresqltutorial.com/).

The schema (the set of tables, their columns and types, and the relationships between them) is diagramed here: http://www.postgresqltutorial.com/postgresql-sample-database.  

We can also get information directly from the database itself.  These commands are specific to each database system.  The commands below are for PostgreSQL.  

First, use `\d` to get a list of relations (tables, views, sequences) -- we'll talk about what views and sequences are later.  

```sql
\dvdrental=# \d
                     List of relations
 Schema |            Name            |   Type   |  Owner   
--------+----------------------------+----------+----------
 public | actor                      | table    | postgres
 public | actor_actor_id_seq         | sequence | postgres
 public | actor_info                 | view     | postgres
 public | address                    | table    | postgres
 public | address_address_id_seq     | sequence | postgres
 public | category                   | table    | postgres
 public | category_category_id_seq   | sequence | postgres
 public | city                       | table    | postgres
 public | city_city_id_seq           | sequence | postgres
 public | country                    | table    | postgres
 public | country_country_id_seq     | sequence | postgres
 public | customer                   | table    | postgres
 public | customer_customer_id_seq   | sequence | postgres
 public | customer_list              | view     | postgres
 public | film                       | table    | postgres
 public | film_actor                 | table    | postgres
 public | film_category              | table    | postgres
 public | film_film_id_seq           | sequence | postgres
 public | film_list                  | view     | postgres
 public | inventory                  | table    | postgres
 public | inventory_inventory_id_seq | sequence | postgres
 public | language                   | table    | postgres
 public | language_language_id_seq   | sequence | postgres
 public | nicer_but_slower_film_list | view     | postgres
 public | payment                    | table    | postgres
 public | payment_payment_id_seq     | sequence | postgres
 public | rental                     | table    | postgres
 public | rental_rental_id_seq       | sequence | postgres
 public | sales_by_film_category     | view     | postgres
 public | sales_by_store             | view     | postgres
 public | staff                      | table    | postgres
 public | staff_list                 | view     | postgres
 public | staff_staff_id_seq         | sequence | postgres
 public | store                      | table    | postgres
 public | store_store_id_seq         | sequence | postgres
(35 rows)
```


Or use `\dt` to get a list of just tables:

```sql
dvdrental=# \dt
             List of relations
 Schema |     Name      | Type  |  Owner   
--------+---------------+-------+----------
 public | actor         | table | postgres
 public | address       | table | postgres
 public | category      | table | postgres
 public | city          | table | postgres
 public | country       | table | postgres
 public | customer      | table | postgres
 public | film          | table | postgres
 public | film_actor    | table | postgres
 public | film_category | table | postgres
 public | inventory     | table | postgres
 public | language      | table | postgres
 public | payment       | table | postgres
 public | rental        | table | postgres
 public | staff         | table | postgres
 public | store         | table | postgres
(15 rows)
```


To get information for an individual table, use 

```
\d tablename
```

```sql
dvdrental=# \d actor
                                         Table "public.actor"
   Column    |            Type             |                        Modifiers                         
-------------+-----------------------------+----------------------------------------------------------
 actor_id    | integer                     | not null default nextval('actor_actor_id_seq'::regclass)
 first_name  | character varying(45)       | not null
 last_name   | character varying(45)       | not null
 last_update | timestamp without time zone | not null default now()
Indexes:
    "actor_pkey" PRIMARY KEY, btree (actor_id)
    "idx_actor_last_name" btree (last_name)
Referenced by:
    TABLE "film_actor" CONSTRAINT "film_actor_actor_id_fkey" FOREIGN KEY (actor_id) REFERENCES actor(actor_id) ON UPDATE CASCADE ON DELETE RESTRICT
Triggers:
    last_updated BEFORE UPDATE ON actor FOR EACH ROW EXECUTE PROCEDURE last_updated()
```

There are also other `\d` describe functions, among them:

`\dn`: list schemas, where are collections of objects (tables, functions) grouped together under a common name

`\df`: list functions

You can get a complete list of backslash commands with `\?`.

# `SELECT`

Select is the command we use most often in SQL.  It let's us select data (specified rows and columns) from one or more tables.  Columns are selected by name, rows are selected with conditional statements (values of a particular column meeting some criteria).  

The basic format of a `SELECT` command is 

```sql
SELECT column_1, column_2 
FROM table1;
```

`SELECT` and `FROM` are reserved keywords.  SQL is case-insensitive, but many times you'll see the key terms in all caps.  Note that you use a semicolon `;` to end the statement.  You can also split a SQL statement across multiple lines -- the space between the terms doesn't matter (a new line counts as space).  

Let's start with the customer table.   The columns in the customer table are:

```sql
dvdrental=# \d customer
                                          Table "public.customer"
   Column    |            Type             |                           Modifiers                            
-------------+-----------------------------+----------------------------------------------------------------
 customer_id | integer                     | not null default nextval('customer_customer_id_seq'::regclass)
 store_id    | smallint                    | not null
 first_name  | character varying(45)       | not null
 last_name   | character varying(45)       | not null
 email       | character varying(50)       | 
 address_id  | smallint                    | not null
 activebool  | boolean                     | not null default true
 create_date | date                        | not null default ('now'::text)::date
 last_update | timestamp without time zone | default now()
 active      | integer                     | 
Indexes:
    "customer_pkey" PRIMARY KEY, btree (customer_id)
    "idx_fk_address_id" btree (address_id)
    "idx_fk_store_id" btree (store_id)
    "idx_last_name" btree (last_name)
Foreign-key constraints:
    "customer_address_id_fkey" FOREIGN KEY (address_id) REFERENCES address(address_id) ON UPDATE CASCADE ON DELETE RESTRICT
Referenced by:
    TABLE "payment" CONSTRAINT "payment_customer_id_fkey" FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON UPDATE CASCADE ON DELETE RESTRICT
    TABLE "rental" CONSTRAINT "rental_customer_id_fkey" FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON UPDATE CASCADE ON DELETE RESTRICT
Triggers:
    last_updated BEFORE UPDATE ON customer FOR EACH ROW EXECUTE PROCEDURE last_updated()
```

To select particular columns we can name the columns:

```sql
SELECT customer_id, store_id FROM customer;
```

Generally with default `psql` settings we will get paged output.  Hit space to get more.  Type `q` or control-c to get your database prompt back.

```
 customer_id | store_id 
-------------+----------
         524 |        1
           1 |        1
           2 |        1
           3 |        1
           4 |        2
           5 |        1
           6 |        2
           7 |        1
           8 |        2
           9 |        2

```

The data you get back are called a result set.  

A pipe character `|` delimits columns.  

If we want all of the columns, we can use `*` as shorthand:

```sql
SELECT * FROM customer;
```

```
 customer_id | store_id | first_name  |  last_name   |                  email                   | address_id | activebool | create_date |       last_update       | active 
-------------+----------+-------------+--------------+------------------------------------------+------------+------------+-------------+-------------------------+--------
         524 |        1 | Jared       | Ely          | jared.ely@sakilacustomer.org             |        530 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           1 |        1 | Mary        | Smith        | mary.smith@sakilacustomer.org            |          5 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           2 |        1 | Patricia    | Johnson      | patricia.johnson@sakilacustomer.org      |          6 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           3 |        1 | Linda       | Williams     | linda.williams@sakilacustomer.org        |          7 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           4 |        2 | Barbara     | Jones        | barbara.jones@sakilacustomer.org         |          8 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           5 |        1 | Elizabeth   | Brown        | elizabeth.brown@sakilacustomer.org       |          9 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           6 |        2 | Jennifer    | Davis        | jennifer.davis@sakilacustomer.org        |         10 | t          | 2006-02-14  | 2013-05-26 14:49:45.738 |      1
           7 |        1 | Maria       | Miller       | maria.miller@sakilacustomer.org          |         11 | t          |           
```

## `LIMIT`

Instead of getting all rows, we can specify a limit of the number of rows to retrieve.

```sql
SELECT customer_id, store_id, first_name, last_name 
FROM customer
LIMIT 5;
```

```sql
 customer_id | store_id | first_name | last_name 
-------------+----------+------------+-----------
         524 |        1 | Jared      | Ely
           1 |        1 | Mary       | Smith
           2 |        1 | Patricia   | Johnson
           3 |        1 | Linda      | Williams
           4 |        2 | Barbara    | Jones
(5 rows)
```

Here, we got all of the output on a single page, and we can see the row count output at the end.

The order of the rows is not random, but it is not guaranteed to be in any particular order by default either.  

## `WHERE`

Instead of getting all rows or a specific number of rows, we can also specify which rows we want by specifying conditions on the values of particular columns (e.g. equals, greater than, less than).

For example, we can select rows from `customer` that have a `store_id=2` with:

```sql
SELECT * FROM customer WHERE store_id=2;
```

(_Note: Going forward, output will only be included when there's something about it to discuss._)

You can combined conditions together with `AND` and `OR`:

```sql
SELECT * FROM customer WHERE store_id=2 AND customer_id=400;
```

Note that string (text) values in SQL are surrounded with single quotes:

```sql
SELECT * FROM staff WHERE first_name='Jon';
```

`WHERE` operators include:

| Operator | Description |
|:---:|:---|
| = | Equal |
| > | Greater than |
| < | Less than |
| >= | Greater than or equal |
| <= | Less than or equal |
| <> or != | Not equal |
| AND | Logical operator AND |
| OR | Logical operator OR |



### `BETWEEN`

`BETWEEN` is shorthand for `# <= x <= #`.  The endpoints are inclusive:

```sql
SELECT * FROM film WHERE film_id BETWEEN 1 AND 5;
```
### `IN`

`IN` lets you specify a lot of values that you would otherwise join together with an `OR` statement:

```sql
SELECT * FROM film WHERE film_id IN (3,5,7,9);
```

### `LIKE`

`LIKE` lets you do pattern matching on strings.  The only two pattern characters are `_` for a single character and `%` for any number of characters (including none).  In some implementations of SQL, `LIKE` is case-insensitive.  In PostgreSQL, it is case-sensitive; `ILIKE` is the PostgreSQL case-insensitive version.

Get names of actors that start with A.

```sql
SELECT * FROM actor WHERE first_name LIKE 'A%';
```   

Note that the following will yield no results:

```sql
SELECT * FROM actor WHERE first_name LIKE 'a%';
``` 

But the following is ok:

```sql
SELECT * FROM actor WHERE first_name ILIKE 'a%';
``` 

Get 4 letter names starting with A:

```sql
SELECT * FROM actor WHERE first_name LIKE 'A___';
```

Get names that end with y:

```sql
SELECT * FROM actor WHERE first_name LIKE '%y';
```

Any names with a z in them?

```sql
SELECT * FROM actor WHERE first_name ILIKE '%z%';
```

```sql
dvdrental=# SELECT * FROM actor WHERE first_name ILIKE '%z%';
 actor_id | first_name | last_name |      last_update       
----------+------------+-----------+------------------------
       11 | Zero       | Cage      | 2013-05-26 14:47:57.62
       89 | Charlize   | Dench     | 2013-05-26 14:47:57.62
      121 | Liza       | Bergman   | 2013-05-26 14:47:57.62
(3 rows)
```

See that we have results where Z is the first letter (since % can match 0 characters) as well as results where there's a z in the middle of the name.


### `IS NULL`

Missing data in SQL is `NULL`.  `NULL` values occur where there is no data entered for a specific row and column.  You can test for `NULL` with `IS NULL`:

```sql
SELECT * FROM address WHERE address2 IS NULL;
```

`NULL` is different than an empty string (''):

```sql
SELECT * FROM address WHERE address2 = '';
```

There is also the opposite: `IS NOT NULL`.


## `ORDER BY`

We can determine the order that our results are show in:

```sql
SELECT film_id, title FROM film ORDER BY film_id;
```

By default, sorting is done in ascending (`ASC`) order.  To get the reverse order of sorting, use `DESC` (descending):

```sql
SELECT film_id, title FROM film ORDER BY film_id DESC;
```

This is often useful to combine with `LIMIT`:

```sql
SELECT film_id, title FROM film ORDER BY film_id DESC LIMIT 5;
```

You can order by multiple columns:

```sql
SELECT customer_id, rental_id FROM rental 
ORDER BY customer_id, rental_id;
```


## `DISTINCT`

`DISTINCT` removes duplicate rows from the result.  It comes before the list of column names, and applies to the combination of all columns.

```sql
SELECT DISTINCT customer_id, staff_id FROM payment;
```

```sql
SELECT DISTINCT amount FROM payment ORDER BY amount;
```



## Functions and Arithmetic

There are many common, built-in functions in PostgreSQL.  See PostgreSQL Documentation for a [full list](https://www.postgresql.org/docs/current/static/functions.html) or [list of mathematical functions](https://www.postgresql.org/docs/current/static/functions-math.html).

For example, we can get the minimum or maximum of a column from results:

```sql
SELECT min(amount) FROM payment;
SELECT max(amount) FROM payment;
```

These functions apply to the result set, not the full table: 

```sql
SELECT max(amount) FROM payment WHERE amount < 4;
```

You can also do arithmetic:

```sql
SELECT rental_duration, rental_duration + 1 
FROM film LIMIT 10;
```

Functions need to be used the part of the query where you specify the values (or other values) that you are selecting, not the where clause.  The following doesn't work:

```sql
SELECT customer_id FROM payment WHERE amount = max(amount);
```

```
dvdrental=# SELECT customer_id FROM payment WHERE amount = max(amount);
ERROR:  aggregate functions are not allowed in WHERE
LINE 1: SELECT customer_id FROM payment WHERE amount = max(amount);
                                                       ^
```

To achieve this, you need to use a subquery, which we'll learn about in the next part.  

NOTE: the set of provided functions is not standard across different implementations of SQL, although there are some common core functions.




## `GROUP BY`

`GROUP BY` is used to divide results into groups, where you then apply some summary function to each group.  You will generally select the column you're grouping by, and then a summary function.  The most common operation is counting.  We use `count(*)` to count the number of rows in each group.

```sql
SELECT customer_id, count(*) FROM rental GROUP BY customer_ID;
```

We can use other functions as well.

```sql
SELECT customer_id, avg(amount) FROM payment GROUP BY customer_ID;
```

You can group by multiple columns:

```sql
SELECT customer_id, amount, count(*) FROM payment 
GROUP BY customer_id, amount;
```

Sort the above

```sql
SELECT customer_id, amount, count(*) FROM payment 
GROUP BY customer_id, amount
ORDER BY customer_id, count(*) DESC;
```


All columns in the `SELECT` part of the statement have to be in the `GROUP BY` part, or you'll get an error:

```sql
SELECT customer_id, amount, count(*) FROM payment GROUP BY customer_id;
```

```sql
dvdrental=# SELECT customer_id, amount, count(*) FROM payment GROUP BY customer_id;
ERROR:  column "payment.amount" must appear in the GROUP BY clause or be used in an aggregate function
LINE 1: SELECT customer_id, amount, count(*) FROM payment GROUP BY c...
```

## `HAVING`

`HAVING` is similar to a `WHERE` clause but it applies to the result of a `GROUP BY` operation; `WHERE` applies before data are grouped by `GROUP BY`;

We can get total amount spent by each customer with:

```sql
SELECT customer_id, sum(amount) FROM payment
GROUP BY customer_id;
```

But how do we just get the customers that spent more than $200?

```sql
SELECT customer_id, sum(amount) FROM payment
GROUP BY customer_id
HAVING sum(amount) > 200;
```


## Dates

So far, we've selected numeric values and string values.  There are also other types, with one of the most common of those being dates.  Dates are in the format `YYYY-MM-DD`.  

```sql
SELECT count(*) FROM customer WHERE create_date = '2006-02-14';
```

Timestamps are dates with a time (and possibly timezone) also attached.  


```sql
SELECT rental_date FROM rental WHERE rental_date < '2005-05-25';
```

```sql
dvdrental=# select rental_date from rental where rental_date<'2005-05-25';
     rental_date     
---------------------
 2005-05-24 22:53:30
 2005-05-24 22:54:33
 2005-05-24 23:03:39
 2005-05-24 23:04:41
 2005-05-24 23:05:21
 2005-05-24 23:08:07
 2005-05-24 23:11:53
 2005-05-24 23:31:46
(8 rows)
```

This will get you everything before 2005-05-25 00:00:00.  If you want just the date part of a datetime:

```sql
SELECT rental_date FROM rental 
WHERE date_trunc('day', rental_date) = '2005-05-24';
```

or

```sql
SELECT rental_date FROM rental
WHERE cast(rental_date as date) = '2005-05-24';
```

or 

```sql
SELECT rental_date FROM rental 
WHERE rental_date::date = '2005-05-24';
```

# Comments

Comments in SQL files:

```sql
/* this is a comment; it can 
span multiple lines */

SELECT * FROM actor; /* this is also a comment */

-- this is a single line comment

SELECT * from actor; -- another single line comment
```
