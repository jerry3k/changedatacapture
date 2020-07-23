    
**Oracle configuration**

Using OracleReader requires the following configuration changes in Oracle:
1. enable archivelog, if not already enabled Enabling archivelog:

    * Log in to SQL*Plus as the sys user.

    * Enter the following command:
    ```sql
    select log_mode from v$database;
    ```
   
    * If the command returns ARCHIVELOG, it is enabled. Skip ahead to Enabling supplemental log data (Step 2.).

    * If the command returns NOARCHIVELOG, enter: shutdown immediate

    * Wait for the message ORACLE instance shut down, then enter: startup mount

    * Wait for the message Database mounted, then enter:
    ```sql
    alter database archivelog;
    alter database open;
    ```
    
    * To verify that archivelog has been enabled, enter select log_mode from v$database; again. This time it should return ARCHIVELOG.

2. enable supplemental log data, if not already enabled  

    *Enter the following command: 
    ```sql
    select supplemental_log_data_min, supplemental_log_data_pk from v$database;
    ```
    * If the command returns YES or IMPLICIT, supplemental log data is already enabled. (SUPPLEME stands for supplemental log data, SUP stands for primary key logging.)
  
    * If supplemental log data is disabled, enter:
   ```sql
    ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
    ```
    * To enable primary key logging for all tables in the database enter: 
    ```sql
    ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
    ```
    
    * To activate your changes, enter:
    ```sql
    alter system switch logfile;
    ```
3. set up LogMiner
```sql
create role striim_privs;
grant create session,
  execute_catalog_role,
  select any transaction,
  select any dictionary
  to striim_privs;
grant select on SYSTEM.LOGMNR_COL$ to striim_privs;
grant select on SYSTEM.LOGMNR_OBJ$ to striim_privs;
grant select on SYSTEM.LOGMNR_USER$ to striim_privs;
grant select on SYSTEM.LOGMNR_UID$ to striim_privs;
create user striim identified by ******** default tablespace users;
grant striim_privs to striim;
alter user striim quota unlimited on users;
```
(Where ******** should be replaced with a strong password.)

4. create a user account for OracleReader and grant it the privileges necessary to use LogMiner 