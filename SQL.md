+ [Server Info](#server-info)
+ [Definition](#definition)
+ [DB Info](#db-info)
+ [Configuration](#config)
+ [User](#user)
+ [Schema](#schema)
+ [Catelog](#pg_catelog)
+ [Prvilege](#prvilege)
+ [Inheritance, Partitioning, copy](#inheritance_partitioning_copy)
+ [Backup](#backup)
+ [DQL](#dql)

# server-info

+ **login server**

    ```psql
    psql -U username;
    ```

+ **DB init**

    ```psql
    init db "dir path"
    ```

+ **Start DB**

    ```psql
    pg_ctl -D "db dir" -p portNum
    ```

+ **current DB dir**

    ```psql
    show data_directory;
    ```

+ **start server**

    ```psql
    pg_ctl start;
    ```

+ **stop server**

    ```psql
    pg_ctl stop;
    ```

+ **restart server**

    ```psql
    pg_ctl restart;
    ```

+ **conf**
  + show conf

    ```psql
    show work_mem;
    ```

  + update conf without restart the server

    ```sql
    alter system set work_mem = '5MB';
    select pg_reload_conf();
    ```

+ **cluster info**

    ```psql
    pg_controldata;
    ```

# definition

| DDL         | DML         | DCL         | TCL      | DQL
| ----------- | ----------- | ----------- | -------- | ------
| Create      | Insert      | Grant       | Commit   | Select
| Drop        | Update      | Revoke      | Rollback |
| Alter       | Delete      |             | Savepoint|
| Truncate    | Merge       |             |          |

# db-info

+ **List of existing DB**

    ```sql
    select * from pg_catalog.pg_database ;
    ```

    ```shell
    \l
    ```

+ **Create DB**

    ```sql
    create database DBname owner username;
    ```

+ **Drop DB**

    ```sql
    drop database dbName;
    ```

  + Issues

    > There are 2 other sessions using the database.

    + Solution

        ```sql
        SELECT 
            pg_terminate_backend(pid) 
        FROM 
            pg_stat_activity    
        WHERE 1=1
            -- don't kill my own connection!
            pid <> pg_backend_pid()
            -- don't kill the connections to other databases
            AND datname = 'dbName'  -- dbName
        ;

        drop database dbName;
        ```

# config

+ **list of conf**

    ```sql
    select * from pg_catalog.pg_file_settings ;
    ```

# user

+ list of user

```psql
\du
```

+ create/drop user

```psql
create user username login password 'password';
drop user joe;
```

+ revoke connection (Normal user can't connect to DB)

```psql
revoke connect on database dbName from public;
```

# schema

+ list of schema

```psql
\dn
```

+ create schema

```sql
create schema schemaName;
```

+ drop schema

```sql
drop schema schemaName;
```

# pg_catelog

> DB & settings info

```sql
select now();  -- current date & time with timezone
select current_user ;  -- current user
select current_schema ;  -- current user
select current_database();  -- current DB
select * from pg_postmaster_start_time();  -- time since server is running
select * from pg_catalog.pg_database;  -- list of DB
select * from pg_catalog.pg_stat_database psd ;
select * from pg_catalog.pg_tablespace pt ;
select oprname, oprcode from pg_catalog.pg_operator po ;
select * from pg_catalog.pg_extension pe ;
select * from pg_catalog.pg_available_extensions pae ;
select * from pg_catalog.pg_shadow ps ;  -- list of user
select * from pg_catalog.pg_timezone_names ptn ; -- available timezone
select * from pg_catalog.pg_locks pl ;  -- which query is executing for long time
select * from pg_catalog.pg_tables pt ;
select * from pg_catalog.pg_views pv ;
select * from pg_catalog.pg_settings ps ;
select * from pg_catalog.pg_indexes pi2 ;
```

# prvilege

+ [previlege](https://tableplus.com/blog/2018/04/postgresql-how-to-grant-access-to-users.html)

# inheritance_partitioning_copy

+ inheritance

    > + All columns of parent table is avalilable in child table.
    >
    > + Parent table will show data comming from parent table + child table.

  + create table

    ```sql
    create table child_table inherits(parent_table);
    ```

  + get data

    ```sql
    select * from parent_table;  -- parent_table + child_table data
    select * from only parent_table;  -- only parent_table data
    select * from child_table;  -- only child_table data
    ```

  + update table

    ```sql
    update parent_table set column_name = 'value' -- update both parent & child table data
    update only parent_table set column_name = 'value'  -- update only parent table data
    update child_table set column_name = 'value' -- update child table data
    ```

  + delete table

    ```sql
    drop table parent_table cascade;  -- drop parent_table
    -- or
    -- First, drop child table & then parent_table.

    drop table child_table;  -- drop child_table
    ```

+ partitioning

    > + postgres partitions table via inheritance
    >
    > + There are 2 types of partition: **range** & **list**

  + create child table (partition table)

    ```sql
    create table child_table(check(parent_table_column_name condition)) inherits(parent_table);
    -- "check" is a constraint which applies some conditions before iserting data into table.
    ```

    + more to add ....

+ copy
  + create table with data

    ```sql
    create table table_name as source_table; 
    ```

  + create table with no data

    ```sql
    create table table_name as table source_table with no data; 

    -- or
    create table tableName
    as
    select * from src_table where 1=2;
    ```

# backup

+ coming soon

# dql

+ filtering operator : ```>,>=,<,<=,=,<>/!=,between,in,not in,and,or,is null, is not null, like(case sensitive), ilike(case insensitive), having```
+ math operator: ```count,max,min,sum,avg,pow,sqrt,abs,round,random,ceil,floor etc```

+ condition

    ```sql
    select
      *  -- all col
    from
      tableName 
    where 1=1
      and colName = value ;
    ```

+ limit & offset

  ```sql
  -- offset: how many rows it skips
  select
    colName1
    , colName2
  from
    tableName 
  offset 1 ;

  -- limit: how many rows will be fetched after skipping
  select
    colName1
    , colName2
  from
    tableName 
  limit 10 offset 1 ;
  ```

+ unique val

  ```sql
  select
      distinct colName1
  from
    tableName ;
  ```

+ search text exists or not

  ```sql
  -- check pattern anywhere in the text
  select
    *
  from
    tableName 
  where 1=1
    and colName1 ilike '%bi%' ;
  
  -- check pattern at the beginning in the text
  select
    *
  from
    tableName 
  where 1=1
    and colName1 ilike 'bi%' ;
  
  -- check pattern at the end in the text
  select
    *
  from
    tableName 
  where 1=1
    and colName1 ilike '%bi' ;
  
  -- alternate of ilike
  select
    *
  from
    tableName 
  where 1=1
    and colName1 ~* 'bi' ;
  
   -- alternate of like
  select
    *
  from
    tableName 
  where 1=1
    and colName1 ~ 'Bi' ;
  ```

+ join

  + inner join/join

    ```sql
    -- if values are same in both col
    select 
      t1.colName1 
      , t1.colName2 
      , t2.colName3 
    from tableName1 as t1 
    join tableName2 as t2 on t1.colName = t2.colName ;
    ```
  
  + left join

    ```sql
    -- all value from left col & non-matched values are null in right col
    select 
      t1.colName1 
      , t1.colName2 
      , t2.colName3 
    from tableName1 as t1 
    left join tableName2 as t2 on t1.colName = t2.colName ;
    ```
  
  + right join

    ```sql
    -- all value from right col & non-matched values are null in left col
    select 
      t1.colName1 
      , t1.colName2 
      , t2.colName3 
    from tableName1 as t1 
    right join tableName2 as t2 on t1.colName = t2.colName ;
    ```
  
  + full join / full outer join

    ```sql
    -- full join: inner join + additional records from left table + additional records from right table
    select 
      t1.colName1 
      , t1.colName2 
      , t2.colName3 
    from tableName1 as t1 
    full join tableName2 as t2 on t1.colName = t2.colName ;
    ```
  
  + cross join

    ```sql
    -- cross join (cartesian product): left records x right records
    select 
      t1.colName1 
      , t2.colName2 
    from tableName1 as t1 
    cross join tableName2 as t2;
    ```
  
  + natural join

    ```sql
    -- sql will decide based on similiar col from both tables
    select 
      t1.colName1 
      , t2.colName2 
    from tableName1 as t1 
    natural join tableName2 as t2;
    ```
  
  + self join

    ```text
    sample input:

    +----+-------+--------+-----------+
    | id | name  | salary | managerId |
    +----+-------+--------+-----------+
    | 1  | Joe   | 70000  | 3         |
    | 2  | Henry | 80000  | 4         |
    | 3  | Sam   | 60000  | Null      |
    | 4  | Max   | 90000  | Null      |
    +----+-------+--------+-----------+

    sample output:

    Output: 
    +----------+
    | Employee |
    +----------+
    | Joe      |
    +----------+
    ```

    ```sql
    select
      em.name as Employee
    from 
      Employee as em
    join 
      Employee as ma on em.managerId = ma.id
    where 1=1
        and em.salary > ma.salary
    ```

+ with clause

  ```sql
  with avg_salry as (
    select
      avg(colName) as salary
    from tableName
  )
  select
    salary as real_salary
  from table1 as t1, avg_salry as t2
  where t1.salary > t2.salary ;
  ```

+ group by

  ```sql
  select  
    t1.colName 
    , count(t2.colName)
  from tableName1 as t1 
  join tableName2 as t2 on t1.colName = t2.colName 
  group by 1 
  order by 1 ; -- sorting (asc, desc): by default asc

  -- condition on group by
  -- ex: count more than 3
  select  
    t1.colName 
    , count(t2.colName)
  from tableName1 as t1 
  join tableName2 as t2 on t1.colName = t2.colName 
  group by 1 
  having count(t2.colName) > 3;
  ```

+ date

  ```sql
  select current_timestamp ;  -- current date with time
  select current_date ;   -- current date
  select current_date + interval '3 hours/minutes/seconds/days/months/years/decades';  -- next
  select current_date - interval '3 hours/minutes/seconds/days/months/years/decades';  -- previous
  select dateCol1 - dateCol2;  -- diff between 2 columns
  select dataCol::date;  -- convert to date type
  ```

  + [date_truc](https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-date_trunc/)

    ```sql
      select 
        date_trunc('year', dateCol)  -- ex: 2023-01-01
        date_trunc('month', dateCol)  -- ex: 2023-02-01, 2023-03-01
        date_trunc('day', dateCol)  -- ex: 2023-02-13, 2023-02-14 
        date_trunc('week', '2023-02-13'::date)  -- ex: 2023-02-13(1st day of the week, starts from monday)
      from 
        tablename;
      ```

  + [date_part](https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-date_part/)

    ```sql
    -- ex: 2023-02-13
    select 
      date_part('year', dateCol)  -- ex: 2023
      date_part('month', dateCol)  -- ex: 02
      date_part('day', dateCol)  -- ex: 13
      date_part('hour', dateCol)  -- ex: hour 24 hour format
      date_part('minute', dateCol)  -- ex: minute
      date_part('second', dateCol)  -- ex: sec --> 12.62
    from 
      tablename;
    ```
  
  + [to_char](https://www.postgresqltutorial.com/postgresql-string-functions/postgresql-to_char/)
  + [to_date](https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-to_date/)
  + [extract](https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-extract/)

  + Unix Timestamp: starts from 1970-01-01

    ```sql
    select 
      extract(epoch from colName) 
    from 
      tableName ;
    ```
