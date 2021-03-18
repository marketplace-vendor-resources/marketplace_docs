## Oracle

launch it: `docker run -d --name oracle -p 1521:1521 -p 5500:5500 -e ORACLE_PWD=oracle oracle/database:18.4.0-xe`
once the db is up: `docker exec -it oracle sqlplus sys/oracle@//localhost:1521/XE as sysdba`
then in SQL console:
`alter session set container = XEPDB1;
CREATE BIGFILE TABLESPACE confluence DATAFILE 'confluence.dbf' SIZE 20M AUTOEXTEND ON;
create user confuser identified by confuser default tablespace confluence quota unlimited on confluence;
grant connect to confuser;
grant resource to confuser;
grant create table to confuser;
grant create sequence to confuser;
grant create trigger to confuser;`
the ‘Service Name’ name for the DV setup wizard will then be XEPDB1

## MS SQL Server

`docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=<PASSWORD>' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2019-latest`

Username to access this will be `sa`

Restoring data:

`sudo docker exec -it SQLServer19 /opt/mssql-tools/bin/sqlcmd    -S localhost -U SA    -Q 'RESTORE DATABASE WideWorldImporters FROM DISK = "/var/opt/mssql/backup/WideWorldImporters-Full.bak" WITH MOVE "WWI_Primary" TO "/var/opt/mssql/data/WideWorldImporters.mdf", MOVE "WWI_UserData" TO "/var/opt/mssql/data/WideWorldImporters_userdata.ndf", MOVE "WWI_Log" TO "/var/opt/mssql/data/WideWorldImporters.ldf", MOVE "WWI_InMemory_Data_1" TO "/var/opt/mssql/data/WideWorldImporters_InMemory_Data_1"'`

Running SQL commands:

`sudo docker exec -it SQLServer19 /opt/mssql-tools/bin/sqlcmd    -S localhost -U SA     -Q 'SELECT Name FROM sys.Databases'`
