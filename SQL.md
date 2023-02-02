+ [Server Info](#server-info)
+ [DB Info](#db-info)
+ [Configuration](#config)
+ [User](#user)
+ [Schema](#schema)
+ [Catelog](#pg_catelog)
+ [Prvilege](#prvilege)
+ [Inheritance, Partitioning, copy](#inheritance_partitioning_copy)
+ [Backup](#backup)

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
    ```

# backup
