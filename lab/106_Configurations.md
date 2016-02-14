# Day 1. Configuration Hands-on Lab
```
1. Change Configuration
 - Cluster Level Configuration
 - DB Level Configuration
 - User Level Configuration
 - session Level Configuration
2. Apply Changes
```

## 1. Change Configuration
#### 1.1. Cluster Level Configuration
```sql
-bash-4.1$ psql
psql.bin (9.5.0.5)
Type "help" for help.

edb=# show work_mem ;
 work_mem
----------
 6698kB
(1 row)

edb=# \q
-bash-4.1$
-bash-4.1$ vi $PGDATA/postgresql.conf

#work_mem = 4MB                         # min 64kB
work_mem = 8MB                          # min 64kB

-bash-4.1$
-bash-4.1$ pg_ctl reload
server signaled
-bash-4.1$
-bash-4.1$ psql
psql.bin (9.5.0.5)
Type "help" for help.

edb=# show work_mem ;
 work_mem
----------
 8MB
(1 row)

edb=#
edb=#
edb=# select * from pg_settings where name='work_mem';
-[ RECORD 1 ]---+----------------------------------------------------------------------------------------------------------------------
name            | work_mem
setting         | 8192
unit            | kB
category        | Resource Usage / Memory
short_desc      | Sets the maximum memory to be used for query workspaces.
extra_desc      | This much memory can be used by each internal sort operation and hash table before switching to temporary disk files.
context         | user
vartype         | integer
source          | configuration file
min_val         | 64
max_val         | 2147483647
enumvals        |
boot_val        | 4096
reset_val       | 8192
sourcefile      | /opt/PostgresPlus/9.5AS/data/postgresql.conf
sourceline      | 127
pending_restart | f

edb=#
```

#### 1.2. DB Level Configuration
```sql
-bash-4.1$ psql
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=#
edb=# \c postgres
You are now connected to database "postgres" as user "enterprisedb".
postgres=# show work_mem ;
 work_mem
----------
 8MB
(1 row)

postgres=#
postgres=# alter database postgres set work_mem = '4MB';
ALTER DATABASE
postgres=#
postgres=# show work_mem ;
 work_mem
----------
 8MB
(1 row)

postgres=# \c postgres
You are now connected to database "postgres" as user "enterprisedb".
postgres=#
postgres=# show work_mem ;
 work_mem
----------
 4MB
(1 row)

postgres=#
postgres=#
postgres=# select * from pg_db_role_setting ;
 setdatabase | setrole |   setconfig
-------------+---------+----------------
       14792 |       0 | {work_mem=4MB}
(1 row)

postgres=# select oid, datname from pg_database ;
  oid  |  datname
-------+-----------
     1 | template1
 14787 | template0
 14792 | postgres
 14793 | edb
(4 rows)

postgres=#
```

#### 1.3. User Level Configuration
```sql
postgres=# create user test password 'ppas';
CREATE ROLE
postgres=#
postgres=# alter user test set work_mem = '6MB';
ALTER ROLE
postgres=# \c postgres test
Password for user test:
You are now connected to database "postgres" as user "test".
postgres=>
postgres=> show work_mem ;
 work_mem
----------
 6MB
(1 row)

postgres=> select * from pg_db_role_setting ;
 setdatabase | setrole |   setconfig
-------------+---------+----------------
       14792 |       0 | {work_mem=4MB}
           0 |   16657 | {work_mem=6MB}
(2 rows)

postgres=> select usename, usesysid from pg_user;
   usename    | usesysid
--------------+----------
 enterprisedb |       10
 test         |    16657
(2 rows)

postgres=>
```

#### 1.4. Session Level Configuration
```sql
postgres=> alter session set work_mem = '10MB';
ALTER SESSION
postgres=>
postgres=> show work_mem ;
 work_mem
----------
 10MB
(1 row)

postgres=>

-- Another session
-bash-4.1$ psql postgres test
Password for user test:
psql.bin (9.5.0.5)
Type "help" for help.

postgres=> show work_mem ;
 work_mem
----------
 6MB
(1 row)

postgres=>
```

## 2. Apply Changes
#### 2.1. pg_ctl
##### 2.1.1. Reload
```
-bash-4.1$ pg_ctl -D $PGDATA reload
server signaled
-bash-4.1$
```
##### 2.1.2. Re-start
```
-bash-4.1$ pg_ctl -D $PGDATA restart -mf -w
waiting for server to shut down.... done
server stopped
waiting for server to start....2016-02-14 12:02:21 KST LOG:  redirecting log output to logging collector process
2016-02-14 12:02:21 KST HINT:  Future log output will appear in directory "pg_log".
 done
server started
-bash-4.1$
```
#### 2.2. psql Command
```sql
-bash-4.1$ psql
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=# \df *reload*
                               List of functions
   Schema   |      Name      | Result data type | Argument data types |  Type
------------+----------------+------------------+---------------------+--------
 pg_catalog | pg_reload_conf | boolean          |                     | normal
(1 row)

edb=#
edb=# select pg_reload_conf;
 pg_reload_conf
----------------
 t
(1 row)

edb=#
```