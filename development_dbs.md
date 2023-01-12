These commands can be used to launch databases on docker.

## PostgreSQL

```
docker run -p 5432:5432 --name some-postgres -e POSTGRES_DB=confluence -e POSTGRES_USER=confuser -e POSTGRES_PASSWORD=confuser -d postgres
```

## MySQL

```
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=confluence -e MYSQL_USER=confuser -e MYSQL_PASSWORD=confuser -p 3306:3306 -p 33060:33060 mysql:8 --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --default-storage-engine=INNODB --max_allowed_packet=256M --innodb_log_file_size=2GB --transaction-isolation=READ-COMMITTED --binlog-format=ROW
```
## Oracle

Launch it:
```
docker run -d --name oracle -p 1521:1521 -p 5500:5500 -e ORACLE_PWD=oracle oracle/database:18.4.0-xe
```

Once the DB is up: 
```
docker exec -it oracle sqlplus sys/oracle@//localhost:1521/XE as sysdba
```

Then in SQL console:
```
alter session set container = XEPDB1;
CREATE BIGFILE TABLESPACE confluence DATAFILE 'confluence.dbf' SIZE 20M AUTOEXTEND ON;
create user confuser identified by confuser default tablespace confluence quota unlimited on confluence;
grant connect to confuser;
grant resource to confuser;
grant create table to confuser;
grant create sequence to confuser;
grant create trigger to confuser;
```
The ‘Service Name’ name for the DV setup wizard will then be `XEPDB1`

## MS SQL Server

```
docker run --name sqlserver -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2017-latest
```

Username to access this will be `sa`

Running SQL commands:

```
docker exec -it SQLServer19 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -Q 'SELECT Name FROM sys.Databases'
```

or

```
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'yourStrong(!)Password'
``` 

And then enter commands like so:

```
CREATE LOGIN confuser WITH PASSWORD = 'Confuser1';
CREATE DATABASE confluence;
ALTER DATABASE confluence COLLATE SQL_Latin1_General_CP1_CS_AS;
ALTER DATABASE confluence SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;
USE confluence;
CREATE USER confuser FOR LOGIN confuser;
GRANT CONTROL TO confuser;
GO
```

Restoring data:

```
docker exec -it SQLServer19 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -Q 'RESTORE DATABASE WideWorldImporters FROM DISK = "/var/opt/mssql/backup/WideWorldImporters-Full.bak" WITH MOVE "WWI_Primary" TO "/var/opt/mssql/data/WideWorldImporters.mdf", MOVE "WWI_UserData" TO "/var/opt/mssql/data/WideWorldImporters_userdata.ndf", MOVE "WWI_Log" TO "/var/opt/mssql/data/WideWorldImporters.ldf", MOVE "WWI_InMemory_Data_1" TO "/var/opt/mssql/data/WideWorldImporters_InMemory_Data_1"'
```
