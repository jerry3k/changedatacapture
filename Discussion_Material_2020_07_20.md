**Talend**:

How to set up CDC:
1. Activate the active log mode in the Oracle database.
```sql
connect / as sysdba;
shutdown;
startup exclusive mount;
alter database archivelog;
alter database open;
```
2. Set up CDC in the Oracle database.
```sql
 create tablespace SOURCE datafile '$ORACLE_PATH/oradata/Oracle/SOURCE.dbf' size 50M;
```
```sql
create user source
identified by source
default tablespace SOURCE 
quota unlimited on SOURCE
quota unlimited on SYSTEM
quota unlimited on SYSAUX;
```
```sql
grant connect, create table to source;
grant create tablespace to source;
grant unlimited tablespace to source;
grant select_catalog_role to source;
grant execute_catalog_role to source;
grant create sequence to source;
grant create session to source;
grant dba to source;
grant execute on SYS.DBMS_CDC_PUBLISH to source;
alter user source quota unlimited on users;
```
```sql
create tablespace PUBLISHER datafile '$ORACLE_PATH/oradata/Oracle/PUBLISHER.dbf' size 50M;
```
```sql
create user publisher
identified by publisher
default tablespace PUBLISHER
quota unlimited on PUBLISHER
quota unlimited on SYSTEM
quota unlimited on SYSAUX;
```
```sql
grant connect, create table to publisher;
grant create tablespace to publisher;
grant unlimited tablespace to publisher;
grant select_catalog_role to publisher;
grant execute_catalog_role to publisher;
grant create sequence to publisher;
grant create session to publisher;
grant dba to publisher;
grant execute on SYS.DBMS_CDC_PUBLISH to publisher;
execute DBMS_STREAMS_AUTH.GRANT_ADMIN_PRIVILEGE(GRANTEE=>'publisher');
```
3. Create and give all rights to the source user.
2. Create and give all rights to the publisher.

Xsteam mode available for Oracle v12

**Syniti** 

Video instruction:
https://syniti.wistia.com/medias/ix48wv0edj

Steps to preform within the program:
1. Set up Security (performed by an Administrator)
2. Register Target and Source Data Sources 
3. Create DBMoto® Source Connections 
4. Encrypt Data Source Password (performed by an Administrator)
5. Configure Parameters in Common 
6. Set up Connection Types 
7. Set up Schedule Groups

**Streamsets**

The following tasks required:
1. Enable LogMiner.
2. Enable supplemental logging for the database or tables.
3. Create a user account with the required roles and privileges to access LOGMNR_LOGS
4. To use the dictionary in redo logs, extract the Log Miner dictionary.
5. Install the Oracle JDBC driver.

**Striim** 

Free plan for smaller organizations and nonproduction workloads. Standard plans range from $100 to $1,250 per month

https://www.striim.com/docs/en/installation-and-configuration-guide.html

Striim supports Oracle 11g 11.2.0.4, 12c 12.1.0.2, 12.2.0.1.0, 18c, and 19c.

    * RAC is supported in all versions.

    * LogMiner is supported in all versions.
    
    *  XStream Out is supported only for 11g on Linux and requires Oracle patches 6880880 & 21352635.

You may choose either LogMiner or XStream Out as the source for change data.
For more info see
https://www.striim.com/docs/en/oracle-database.html

Oracle configuration

Using OracleReader requires the following configuration changes in Oracle:

1. enable archivelog, if not already enabled https://www.striim.com/docs/en/enabling-archivelog.html

2. enable supplemental log data, if not already enabled  https://www.striim.com/docs/en/enabling-supplemental-log-data.html

3. set up either LogMiner or XStream Out
https://www.striim.com/docs/en/creating-an-oracle-user-with-logminer-privileges.html

4. create a user account for OracleReader and grant it the privileges necessary to use LogMiner or XStream Out