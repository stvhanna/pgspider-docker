# pgspider docker file

> include v8 extension

## Usegae

* sqlite fdw

```code
CREATE EXTENSION pgspider_core_fdw;
CREATE EXTENSION sqlite_fdw;
CREATE EXTENSION pgspider_fdw;

CREATE SERVER parent FOREIGN DATA WRAPPER pgspider_core_fdw OPTIONS (host '127.0.0.1', port '5432');

CREATE SERVER sqlite_svr FOREIGN DATA WRAPPER sqlite_fdw OPTIONS(database '/opt/proxysql.db');

CREATE USER MAPPING FOR CURRENT_USER SERVER parent OPTIONS(user 'postgres', password 'dalong');

CREATE FOREIGN TABLE mysql_users(username text, password text,default_schema text, __spd_url text) SERVER parent;

CREATE FOREIGN TABLE mysql_users__sqlite_svr__0(username text, password text,default_schema text) SERVER sqlite_svr OPTIONS (table 'mysql_users');

select * from mysql_users;
```

* mongodb fdw

```code
CREATE EXTENSION mongo_fdw;
CREATE EXTENSION pgspider_core_fdw;
CREATE EXTENSION postgres_fdw;
CREATE EXTENSION pgspider_fdw;

CREATE SERVER parent FOREIGN DATA WRAPPER pgspider_core_fdw OPTIONS (host '127.0.0.1', port '5432');

CREATE SERVER mongo_server FOREIGN DATA WRAPPER mongo_fdw OPTIONS (address 'mongo', port '27017', authentication_database 'admin');

CREATE USER MAPPING FOR postgres SERVER mongo_server OPTIONS(username 'dalong', password 'dalong');

CREATE FOREIGN TABLE userapps(_id NAME,appid int,appname text,__spd_url text) SERVER parent;

CREATE FOREIGN TABLE userapps__mongo_server__0(_id NAME,appid int,appname text) SERVER mongo_server OPTIONS (database 'apps', collection 'userapps');

select * from userapps;

mongodb docs:

{
    "_id" : ObjectId("5e3a782b132f94cefe1d1e60"),
    "appname" : "demoapp",
    "appid" : 1
}
```

## pg fdw

```code
CREATE EXTENSION pgspider_core_fdw;
CREATE EXTENSION postgres_fdw;
CREATE EXTENSION pgspider_fdw;

CREATE SERVER parent FOREIGN DATA WRAPPER pgspider_core_fdw OPTIONS (host '127.0.0.1', port '5432');

CREATE SERVER postgres_svr FOREIGN DATA WRAPPER postgres_fdw OPTIONS(host 'pg', port '5432', dbname 'postgres');

CREATE USER MAPPING FOR CURRENT_USER SERVER parent OPTIONS(user 'postgres', password 'dalong');

CREATE USER MAPPING FOR CURRENT_USER SERVER postgres_svr OPTIONS(user 'postgres', password 'dalong');

CREATE FOREIGN TABLE t1(i int, t text, __spd_url text) SERVER parent;

CREATE FOREIGN TABLE t1__postgres_svr__0(i int, t text) SERVER postgres_svr OPTIONS (table_name 't1');

pg datas:
CREATE TABLE t1 (
    i SERIAL PRIMARY KEY,
    t text
);

INSERT INTO "public"."t1"("i","t")
VALUES
(1,E'demo');
```

## mysql fdw

```code
CREATE EXTENSION pgspider_core_fdw;
CREATE EXTENSION mysql_fdw;
CREATE EXTENSION pgspider_fdw;

CREATE SERVER parent FOREIGN DATA WRAPPER pgspider_core_fdw OPTIONS (host '127.0.0.1', port '5432');

CREATE SERVER mysql_svr FOREIGN DATA WRAPPER mysql_fdw OPTIONS(host 'mysql', port '3306');

CREATE USER MAPPING FOR CURRENT_USER SERVER parent OPTIONS(user 'root', password 'dalongrong');

CREATE USER MAPPING FOR CURRENT_USER SERVER mysql_svr OPTIONS(username 'root', password 'dalongrong');

CREATE FOREIGN TABLE apps(id int, appname text, __spd_url text) SERVER parent;
CREATE FOREIGN TABLE apps__mysql_svr__0(id int, appname text) SERVER mysql_svr OPTIONS (dbname 'demo', table_name 'apps');

select * from apps;

mysql db:

CREATE TABLE `apps` (
  `id` bigint(20) DEFAULT NULL,
  `appname` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci


INSERT INTO demo.apps (id,appname) VALUES 
(1,'demo')
;
```

## influxdb fdw

```code
CREATE EXTENSION influxdb_fdw;
CREATE SERVER influxdb_server FOREIGN DATA WRAPPER influxdb_fdw OPTIONS
(dbname 'mydb', host 'http://influxdb', port '8086') ;
CREATE USER MAPPING FOR CURRENT_USER SERVER influxdb_server OPTIONS(user 'dalong', password 'dalong');
CREATE FOREIGN TABLE t1(time timestamp with time zone , age int,name text, email text,user_id int) SERVER influxdb_server OPTIONS (table 'demouser');
SELECT * FROM t1;

or import schema:
IMPORT FOREIGN SCHEMA public FROM SERVER influxdb_server INTO public;
select * from demouser;
insert into influxdb datas:
demouser,name=dalong,age=30 user_id=100,email="dalong@qq.com"
demouser,name=荣锋亮,age=20 user_id=10,email="dalongrong@qq.com"

```
## griddb fdw

```code
CREATE EXTENSION griddb_fdw;

// use notification_member 
CREATE SERVER griddb_svr FOREIGN DATA WRAPPER griddb_fdw OPTIONS(notification_member 'griddb:10001',clustername 'defaultCluster');
CREATE USER MAPPING FOR public SERVER griddb_svr OPTIONS(username 'admin', password 'admin');
IMPORT FOREIGN SCHEMA griddb_schema FROM SERVER griddb_svr INTO public;
```

## sql server fdw

```code

init some datas:
in sql server:
create DATABASE appdemos;
use appdemos;
create table apps (
    id int,
    age int,
    name VARCHAR(256)
);
insert into apps VALUES(1,22,'appdemo');
insert into apps VALUES(2,30,'荣锋亮');
in pg database:

CREATE EXTENSION tds_fdw;
CREATE SERVER mssql_svr
	FOREIGN DATA WRAPPER tds_fdw
	OPTIONS (servername 'db', port '1433', database 'appdemos',msg_handler 'notice',character_set 'UTF-8');

CREATE USER MAPPING FOR postgres
	SERVER mssql_svr 
	OPTIONS (username 'sa', password 'Dalong!123%');

CREATE FOREIGN TABLE apps (
	id integer,
	age integer,
    name varchar)
	SERVER mssql_svr
	OPTIONS (table_name 'dbo.apps', row_estimate_method 'showplan_all');
or  import schema:

IMPORT FOREIGN SCHEMA dbo
	FROM SERVER mssql_svr
	INTO public
	OPTIONS (import_default 'true');

select * from  apps
```

## oracle fdw

```code
CREATE EXTENSION oracle_fdw;
CREATE SERVER oradb FOREIGN DATA WRAPPER oracle_fdw
          OPTIONS (dbserver '//host:1521/sjzl');
          CREATE USER MAPPING FOR postgres SERVER oradb
          OPTIONS (user 'user', password 'password');
        
          CREATE FOREIGN TABLE <map-table> (
          PK_CASE        VARCHAR,
          CREATEUSER     VARCHAR,
          APPLICANTID  VARCHAR
       ) SERVER oradb OPTIONS (schema 'ORAUSER', table 'table');

  CREATE FOREIGN TABLE <map-table> (
          CLIENTID        VARCHAR,
          CLIENTNAME     VARCHAR,
          PK_CLIENT  VARCHAR
       ) SERVER oradb OPTIONS (schema 'ORAUSER', table 'table');
       select * from <<map-table>>;
```