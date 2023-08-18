+ [Page](#page)
+ [Server Info](#server-info)
+ [Definition](#definition)
+ [DB Info](#db-info)
+ [Configuration](#config)
+ [User](#user)
+ [Schema](#schema)
+ [Catelog](#pg_catelog)
+ [Previlege](#previlege)
+ [Inheritance, Partitioning, copy](#inheritance_partitioning_copy)
+ [Backup](#backup)
+ [DDL](#ddl)
+ [DQL](#dql)
+ [View And Materialized view](#view_and_materialized_view)
+ [Keys/Constraints](#keys)
+ [Normalization](#normalization)
+ [Data Modeling](#data_modeling)
+ [CMD](#cmd)

# page

+ page is a smallest unit of data stroage
+ every table & index is stored as an array of pages of fixed size
+ By default, in PostgreSQL page is 8kb

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

  + <span style='color: orange'>**Types of shutdown**</span>
    + <span style='color: yellow'>**smart**</span> : It disallows new connection but lets existing sessions end the transaction normally. It shutdows after all session terminated.
    + <span style='color: yellow'>**fast**</span> : It disallows new connection and abort their connection & exits gracefully(when the server restarts it doesn't need recover data)
    + <span style='color: yellow'>**immediate**</span> : Quits/aborts with proper shutdown which lead to recovery on next startup.

    ```psql
    pg_ctl stop;
    ```

+ **restart and reload server**

    ```psql
    pg_ctl restart;
    pg_ctl reload;
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

  ```sql
  create user username login password 'password';  -- new user
  drop user joe;  -- delete user
  alter user username with password 'password';  -- update user password
  ```

+ revoke connection (Normal user can't connect to DB)

  ```sql
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

# previlege

+ [previlege](https://tableplus.com/blog/2018/04/postgresql-how-to-grant-access-to-users.html)

# inheritance_partitioning_copy

## **<span style="color: orange">inheritance</span>**

  > + All columns of parent table is avalilable in child table.
  >
  > + Parent table will show data comming from parent table + child table.

+ <span style="color: yellow">create table</span>

    ```sql
    create table child_table inherits(parent_table);
    ```

+ <span style="color: yellow">get data</span>

    ```sql
    select * from parent_table;  -- parent_table + child_table data
    select * from only parent_table;  -- only parent_table data
    select * from child_table;  -- only child_table data
    ```

+ <span style="color: yellow">update table</span>

    ```sql
    update parent_table set column_name = 'value' -- update both parent & child table data
    update only parent_table set column_name = 'value'  -- update only parent table data
    update child_table set column_name = 'value' -- update child table data
    ```

+ <span style="color: yellow">delete table</span>

    ```sql
    drop table parent_table cascade;  -- drop parent_table
    -- or
    -- First, drop child table & then parent_table.

    drop table child_table;  -- drop child_table
    ```

## <span style="color: orange">partitioning</span>

> + postgres partitions table via inheritance
>
> + There are 2 types of partition : **range** & **list**

+ <span style="color: yellow">create child table (partition table)</span>

    ```sql
    create table child_table(check(parent_table_column_name condition)) inherits(parent_table);
    -- "check" is a constraint which applies some conditions before iserting data into table.
    ```

+ <span style="color: yellow">function</span>

  ```sql
  CREATE OR REPLACE FUNCTION public.fn_name()
  RETURNS trigger
  LANGUAGE plpgsql
  AS $function$
  begin
    if (extract(year from new.parentTableCol) >= 2015 and extract(year from new.parentTableCol) < 2016) then
    insert into public.childTable1 values (new.*) ;
    elsif (extract(year from new.parentTableCol) >= 2016 and extract(year from new.parentTableCol) < 2017) then
    insert into public.childTable2 values (new.*) ;
    elsif (extract(year from new.parentTableCol) >= 2017 and extract(year from new.parentTableCol) < 2018) then
    insert into public.childTable3 values (new.*) ;
    else
    raise exception 'date must be less than 2018' ;
    end if ;
    return null ;
  end ;
  $function$ ;
  ```

+ <span style="color: yellow">trigger</span>

  ```sql
  create trigger triggerName before insert on public.childTable for each row execute procedure public.fn_name() ;
  ```

+ <span style="color: yellow">copy</span>
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

# ddl

+ change column data type
  + date

    ```sql
    alter table tableName alter column columnName type date using columnName::date ;
    ```

+ change table name

    ```sql
    ALTER TABLE IF EXISTS tableName RENAME TO newTableName ;
    ```

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

+ filter

    ```text
    sample input:

    titanic table:         
    +----------+--------+                          
    | survived | pclass |             
    +----------+--------+             
    | 0        | 3      |             
    | 1        | 1      |             
    | 1        | 2      |             
    +----------+--------+               

    sample output:

    Output: 
    +----------+------------+-------------+----------+                          
    | survived | first_class|second_class |third_class|          
    +----------+------------+-------------+----------+             
    | 0        | 11         | 6           | 42             
    | 1        | 10         | 12          | 19                    
    +----------+------------+-------------+----------+
    ```

    ```sql
    select
      survived
      , count(1) filter(where pclass=1) as first_class
      , count(1) filter(where pclass=2) as second_class
      , count(1) filter(where pclass=3) as third_class
    from titanic
    group by 1
    ;
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

    Ex (inner, left, right)

    ```text
    sample input:

    src table:                   tgt table:
    +----+-------+               +----+-------+             
    | id | name  |               | id | name  |
    +----+-------+               +----+-------+
    | 1  | a     |               | 1  | a     |
    | 2  | b     |               | 2  | b     |
    | 3  | c     |               | 4  | x     |
    | 4  | c     |               | 5  | f     |
    +----+-------+               +----+-------+

    sample output:

    Output: 
    +----------+-----------+
    | id       |name       |
    +----------+-----------+
    | 3        | new in src|
    | 4        | mismatch  |
    | 5        | new in tgt|
    +----------+-----------+
    ```

    ```sql
    select
      aa.id
      , 'mismatch' as comment
    from
      public.src aa
    join public.tgt bb on aa.id = bb.id and aa.name <> bb.name
    union 
    select
      aa.id
      , 'new in src' as comment
    from
      public.src aa
    left join public.tgt bb on aa.id = bb.id
    where bb.id is null
    union 
    select
      bb.id
      , 'new in tgt' as comment
    from
      public.src aa
    right join public.tgt bb on aa.id = bb.id
    where aa.id is null ;
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

    ```text
    sample input:

    src table:                   tgt table:
    +----+-------+               +----+-------+             
    | id | name  |               | id | name  |
    +----+-------+               +----+-------+
    | 1  | a     |               | 1  | a     |
    | 2  | b     |               | 2  | b     |
    | 3  | c     |               | 4  | x     |
    | 4  | c     |               | 5  | f     |
    +----+-------+               +----+-------+

    sample output:

    Output: 
    +----------+-----------+
    | id       |name       |
    +----------+-----------+
    | 3        | new in src|
    | 4        | mismatch  |
    | 5        | new in tgt|
    +----------+-----------+
    ```

    ```sql
    with cte as
    (
      select
        aa.id as a_id
        , aa.name as a_name
        , bb.id as b_id
        , bb.name as b_name
      from
        public.src aa
      full join public.tgt bb on aa.id = bb.id
    ),
    conditions as
    (
      select
        a_id
        , b_id
        , case
          when b_id is null then 'new in src'
          when a_id is null then 'new in tgt'
          when a_id = b_id and a_name <> b_name then 'mismatch'
        end as result
      from cte
    )
    select
      coalesce(a_id, b_id) as id
      , result
    from conditions
    where 1=1
      and result is not null ;
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

+ set
  + must be same size of columns
  + union

    ```sql
    -- without duplicate
    select col1, col2 from tableA
    union
    select col1, col2 from tableB
    ```

  + union all

    ```sql
    -- with duplicate
    select col1, col2 from tableA
    union all
    select col1, col2 from tableB
    ```
  
  + intersect

    ```sql
    -- distinct rows that appeared in both input query
    select col1, col2 from tableA
    intersect
    select col1, col2 from tableB
    ```

  + except

    ```sql
    -- rows that appeared in first query but not in second query
    select col1, col2 from tableA
    except
    select col1, col2 from tableB
    ```
  
  + precedence

    ```mermaid
    flowchart LR

    1.parentheses--> 2.intersect--> 3.union/except
    ```

+ window fn

  ```text
  # ranking fn: row_number,rank,dense_rank,ntile
  # offset window fn: lead,lag,first_value, last_value
  # others: nth_value
  ```

  + row_number

    ```text
    Problem: The Latest Login in 2020
    =================================
    Input: 

    Logins table:
    +---------+---------------------+
    | user_id | time_stamp          |
    +---------+---------------------+
    | 6       | 2020-06-30 15:06:07 |
    | 6       | 2021-04-21 14:06:06 |
    | 6       | 2019-03-07 00:18:15 |
    | 8       | 2020-02-01 05:10:53 |
    | 8       | 2020-12-30 00:46:50 |
    | 2       | 2020-01-16 02:49:50 |
    | 2       | 2019-08-25 07:59:08 |
    | 14      | 2019-07-14 09:00:00 |
    | 14      | 2021-01-06 11:59:59 |
    +---------+---------------------+

    Output: 
    +---------+---------------------+
    | user_id | last_stamp          |
    +---------+---------------------+
    | 6       | 2020-06-30 15:06:07 |
    | 8       | 2020-12-30 00:46:50 |
    | 2       | 2020-01-16 02:49:50 |
    +---------+---------------------+
    ```

    ```sql
    select
      t1.user_id
      , t1.last_stamp
    from
    (select
        user_id
        ,time_stamp as last_stamp
        , row_number() over(partition by user_id order by time_stamp desc) as rnk
    from Logins 
    where 1=1 
        and year(time_stamp) = 2020) t1
    where 1=1
        and t1.rnk = 1 ;
    ```

  + dense_rank and rank

    ```text
    Problem: Rank Scores
    ======================

    Input:

    Scores table:
    +----+-------+
    | id | score |
    +----+-------+
    | 1  | 3.50  |
    | 2  | 3.65  |
    | 3  | 4.00  |
    | 4  | 3.85  |
    | 5  | 4.00  |
    | 6  | 3.65  |
    +----+-------+

    Output:
    +-------+------+
    | score | rank |
    +-------+------+
    | 4.00  | 1    |
    | 4.00  | 1    |
    | 3.85  | 2    |
    | 3.65  | 3    |
    | 3.65  | 3    |
    | 3.50  | 4    |
    +-------+------+
    ```

    ```sql
    -- dense_rank: it doesn't skip any seq for same val
    select
      score
      , DENSE_RANK() over(order by score desc) as "rank"
    from Scores ;

    -- rank: it skips the seq for same val
    select
      score
      , RANK() over(order by score desc) as "rank"
    from Scores ;
    ```

  + lag: ```previoius record```

    ```sql
    -- dept wise emp previous salary
    select
      emp_name
      , salary
      , lag(salary) over(partition by dep order by emp_id) as seq
    
    -- dept wise emp 1st two previous salary
    select
      emp_name
      , salary
      , lag(salary,2,0) over(partition by dep order by emp_id) as seq  -- 1st:col, 2nd:how many previous row, 3rd: if null then 0 or default val
    ```
  
  + lead: ```next record```

    ```sql
    -- dept wise emp next salary
    select
      emp_name
      , salary
      , lead(salary) over(partition by dep order by emp_id) as seq
    
    -- dept wise emp 1st two next salary
    select
      emp_name
      , salary
      , lead(salary,2,0) over(partition by dep order by emp_id) as seq  -- 1st:col, 2nd:how many previous row, 3rd: if null then 0 or custom val
    ```
  
  + first_value : It returns the 1st row

    ```sql
    select
      colName1  
    , colName2 
    , colName3 
    , first_value ("colName") over(partition by colName order by colName desc) as max_price_product_name
    from
      tableName ;
    ```
  
  + last_value : It returns the last row

    ```sql
    select
      colName1  
    , colName2 
    , colName3 
    , last_value (colName) over(partition by colName order by colName desc range between unbounded preceding and unbounded following) as min_price_product_name
    from
      tableName ;
    ```
  
  + nth_value

    ```sql
    -- 3rd highest
    select
       colName1  
      , colName2 
      , colName3    
      , nth_value(colName, 3) over(partition by colName order by ProductPrice desc) as seq
    from
      tableName ;
    ```
  
  + ntile: group the whole data into certain buckets

    ```sql
    -- grouping the whole data based on certain col
    select
       colName1  
      , colName2 
      , colName3
      ntile(3) over(partition by colName order by "ProductPrice" desc) as bucket
    from
      tableName ;

    -- grouping the whole data
    select
       colName1  
      , colName2 
      , colName3
      ntile(3) over(order by "ProductPrice" desc) as bucket
    from
      tableName ;
    ```

    Ex:

      ```text
      sample input:

      +----+-------+
      | id | name  |
      +----+-------+
      | 1  | emp1  |
      | 2  | emp2  |
      | 3  | emp3  |
      | 4  | emp4  |
      | 5  | emp5  |
      | 6  | emp6  |
      | 7  | emp7  |
      | 8  | emp8  |
      +----+-------+

      sample output:

      Output: 
      +-----------------+
      | Employee        |
      +-----------------+
      | 1 emp1, 2 emp2  |
      | 3 emp3, 4 emp4  |
      | 5 emp5, 6 emp6  |
      | 7 emp7, 8 emp8  |
      +-----------------+
      ```

      ```sql
      -- solution:
      -- =========
      with cte as
      (
        select
          concat(id::text,  ' ',name) as idn
          , ntile(4) over(order by id) as grp
        from
          tableName
      ),
      grouping as
      (
        select
          grp
          , string_agg(idn,', ') as result
        from 
          cte
        group by 1
        order by 1
      )
      select
        result
      from 
        grouping ;  
      ```
  
  + cume_dist: the percentage of each row compared to whole dataset.

    ```sql
    -- top 30% products
    select 
      t1.*
    from
    (
      select
        "ProductPrice"::int ,
        round(cume_dist() over(order by "ProductPrice" desc)::numeric*100, 2) as cumee_dist
      from
       tableName
    ) t1
    where 1=1
      and t1.cumee_dist <= 30 ;
    ```
  
  + percent_rank: relative rank in percentage compared to others

    ```sql
    select
      "ProductPrice"::int ,
      round(percent_rank() over(order by "ProductPrice")::numeric*100, 2) as per_rank
    from
      tableName
    ```
  
  + Ex
    + cumulative sum

      ```text
        sample input:
        
        activity

        +----+--------------+
        | player_id | game  | 
        +-----------+-------+
        | 1         | 5     |
        | 1         | 6     |
        | 2         | 1     |
        | 3         | 0     |
        | 3         | 5     |
        +-----------+-------+

        sample output:

        Output: 
        +----+--------------+
        | player_id | game  | 
        +-----------+-------+
        | 1         | 5     |
        | 1         | 11    |
        | 2         | 1     |
        | 3         | 0     |
        | 3         | 5     |
        +-----------+-------+
        ```

        ```sql
        select
          player_id
          , sum(game) over(partition by player_id order by game) as game
        from
          public.activity 
        order by player_id ;

        -- or,
        
        select
          player_id
          , lag(game,1,0) over(partition by player_id order by game) + game as game
        from
          public.activity 
        order by player_id ;
        ```

# view_and_materialized_view

+ view
  1. It always provide latest data.
  2. It doesn't store data.
  3. Can't change the col name in src code after creating view.
  4. Can't change the order (have to add new col at the end).
  5. Can't change the data type.

    ```sql
    create or replace view viewName
    as
    select
      id
      , avg(val)
      , count(1)
    from tableName
    group by 1 ;

    select * from viewName ;

    -- with check option
    create or replace view viewName
    as
    select
      id
      , salary
      , dept
    from tableName
    where dept = 1
    with check option ;

    /*
    view created by check option
    will restrict inserting unnecessary/wrong data. 
    */
    insert into viewName
    values (1, 10000, 2) ;
    ```

+ materialized view

  1. It doesn't update automatically.
  2. It stores data

    ```sql
    create or replace materialized view viewName
    as
    select
      id
      , avg(val)
      , count(1)
    from tableName
    group by 1 ;

    select * from viewName ;
    ```

  + update materialized view data

    ```sql
    refresh materialized view viewName ;
    ```

# normalization

> It removes/minimizes redundancy.

+ types : ```1NF, 2NF, 3NF, BCNF, 4NF, 5NF```

+ **<span style="color: orange">1NF</span>**
  + Each col should have atomic values
  + A col must be with individual type, not mixed
  + Each col should have unique name
  + order doesn't matter

<br>

+ **<span style="color: orange">2NF</span>**
  + It sould be in 1NF
  + Each non-key column is dependent on the entire primary key. This means that if a table has a **composite primary key(primary key of 1st table = primary key of 2nd table + primary key of 3rd table)**, each non-key column should depend on the entire composite key, not just part of it. **<span style="color: red">In essence, no partial dependency</span>.**

<br>

+ **<span style="color: orange">3NF</span>**
  + It sould be in 2NF
  + No transitive dependency (When any column is dependent on non primary key column)

<br>

+ **<span style="color: orange">BCNF (Boyce-codd normal form)</span>**
  + There should be no dependencies between non-superkey attributes

<br>

+ **<span style="color: orange">4NF</span>**
  + It sould be in BCNF
  + It should not have multi-values dependency ( if a table has a composite key, each non-key attribute should depend only on the entire composite key, not just part of it)

<br>

+ **<span style="color: orange">5NF/PJNF (project join normal form)</span>**
  + It sould be in 4NF
  + It has no join dependencies (all non-key attributes should be fully dependent on the primary key, and not on any other non-key attributes)

# keys

+ super key : An attribute or set of attribute is used to identify a row of data.
+ candidate key: subset of super key
+ primary key: A table must have only one primary key column.

  ```sql
  -- while creating table
  CREATE TABLE tableName (
  col1 serial4 NOT NULL,
  col2 varchar(45) NOT NULL,
  col3 varchar(45) NOT NULL,
  col4 timestamp NOT NULL DEFAULT now(),
  CONSTRAINT col1_pkey PRIMARY KEY (col1)
  );

  -- while updating table
  alter table tableName 
  add constraint col_pk primary key(colName) ;

  ```

+ unique key: A table may have multiple unique column.

  ```sql
  alter table tableName
  add constraint col2_unique unique(colName);

  ```

+ foreign key: it's used to create relationship with another table.

  ```sql
  alter table tableName
  add constraint constraint_name foreign key(colName)
  references primary_table(colName);
  ```

+ check constraint:

  ```sql
  alter table tableName
  add constraint constraint_name check(id > 1);
  ```

+ composite key: any key with more than one attribute
+ compound key: any composite key has at least one foreign key attribute.
+ surrogate key: If a table has no relationship or unique identifier then we create primary key. it's called surrogate key.

# data_modeling

+ **<span style="color:orange">dimension</span>**
+ **<span style="color:orange">fact : dimension key + measure</span>**
+ **<span style="color:orange">schema</span>**
  + **<span style="color:yellow">star</span>** : It's one dimensional table contains most sets of attributes. One fact table related multiple dimension tables.
    + [docs](https://tinyurl.com/ys25xhdz)
  + **<span style="color:yellow">snowflake</span>** : The dimension tables related to the fact table may have another dimension table.
    + [docs](https://tinyurl.com/y854rrz3)
  + **<span style="color:yellow">fact constellation(galaxy)</span>** : It has more than 1 fact table.
    + [docs](https://tinyurl.com/56uz82a4)

# cmd

+ data upload from file to table
  
  ```text
  PGPASSWORD=password psql -h 127.0.0.1 -U dbUser -d DBName -c "\copy schemaName.tableName(colName1, colName2, ..., colName_n) from '/filePath/fileName.csv' header delimeter ',' csv"
  ```
