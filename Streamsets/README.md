Created by: J.Murray\
Date: 1st July 2020
1. - [x] Firewalls configured in Azure 
2. - [x] VMs in Azure with access to OCI and DB2/AS400 (DONE)
3. POC using
   1. - [ ] GoldenGate
   2. - [x] Syniti
   3. - [x] Streamsets (Opensource)
   4. - [x] Talend Cloud Data Integration
4. Final Product Selected
   1. - [x] Syniti - for DB2i and(or) Oracle
   2. - [x] Streamsets - for Oracle CDC
# ORACLE (CDC)
## Option A - Using Oracle LogMiner
The Oracle CDC Client processes change data capture (CDC) information provided by Oracle LogMiner redo logs. Use Oracle CDC Client to process data from Oracle 11g, 12c, 18c, or 19c.
You might use this origin to perform database replication. You can use a separate pipeline with the JDBC Query Consumer or JDBC Multitable Consumer origin to read existing data. Then start a pipeline with the Oracle CDC Client origin to process subsequent changes.

Oracle CDC Client processes data based on the commit number, in ascending order.

To read the redo logs, Oracle CDC Client requires the LogMiner dictionary. The origin can use the dictionary in redo logs or in an online catalog. When using the dictionary in redo logs, the origin captures and adjusts to schema changes. The origin can also generate events when using redo log dictionaries.

The origin can create records for the INSERT, UPDATE, SELECT_FOR_UPDATE, and DELETE operations for one or more tables in a database. You can select the operations that you want to use. The origin also includes CDC and CRUD information in record header attributes so generated records can be easily processed by CRUD-enabled destinations. For an overview of Data Collector changed data processing and a list of CRUD-enabled destinations, see Processing Changed Data.

> **Note**: To use Oracle CDC Client, you must enable LogMiner for the database that you want to use and complete the necessary prerequisite tasks. The origin uses JDBC to access the database.

The following tasks required:
1. Enable LogMiner.
2. Enable supplemental logging for the database or tables.
3. Create a user account with the required roles and privileges to access LOGMNR_LOGS
4. To use the dictionary in redo logs, extract the Log Miner dictionary.
5. Install the Oracle JDBC driver.

## Task 1. Enable LogMiner
LogMiner provides redo logs that summarize database activity. The origin uses these logs to generate records.

LogMiner requires an open database in ARCHIVELOG mode with archiving enabled. To determine the status of the database and enable LogMiner, use the following steps:
1. Log into the database as a user with DBA privileges.
2. Check the database logging mode:
    ```sql
    select log_mode from v$database;
    ```
    If the command returns ARCHIVELOG, you can skip to Task 2.

    If the command returns NOARCHIVELOG, continue with the following steps:

3. Shut down the database:
    ```sql
    shutdown immediate;
    ```
1. Start up and mount the database:
    ```sql
    startup mount;
    ```
1. Configure enable archiving and open the database:
    ```sql
    alter database archivelog;
    alter database open;
    ```
## Task 2. Enable Supplemental Logging
To retrieve data from redo logs, LogMiner requires supplemental logging for the database or tables.

Enable at least primary key or "identification key" logging at a table level for each table that you want to use. With identification key logging, records include only the primary key and changed fields.

Due to an Oracle known issue, to enable supplemental logging for a table, you must first enable minimum supplemental logging for the database.

To include all fields in the records the origin generates, enable full supplemental logging at a table or database level. Full supplemental logging provides data from all columns, those with unchanged data as well as the primary key and changed columns. For details on the data included in records based on the supplemental logging type, see Generated Records.
1. To verify if supplemental logging is enabled for the database, run the following command:
    ```sql
    SELECT supplemental_log_data_min, supplemental_log_data_pk, supplemental_log_data_all FROM v$database;
    ```
If the command returns Yes or Implicit for all three columns, then supplemental logging is enabled with both identification key and full supplemental logging. You can skip to Task 3.

If the command returns Yes or Implicit for the first two columns, then supplemental logging is enabled with identification key logging. If this is what you want, you can skip to Task 3.

2. Enable identification key or full supplemental logging.
For 12c, 18c, or 19c multitenant databases, best practice is to enable logging for the container for the tables, rather than the entire database. You can use the following command first to apply the changes to just the container:
    ```sql
    ALTER SESSION SET CONTAINER=<pdb>;
    ```
You can enable identification key or full supplemental logging to retrieve data from redo logs. You do not need to enable both:
**To enable identification key logging**

You can enable identification key logging for individual tables or all tables in the database:
* For individual tables

    Use the following commands to enable minimal supplemental logging for the database, and then enable identification key logging for each table that you want to use:
    ```sql
    ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
    ```
    ```sql
    ALTER TABLE <schema name>.<table name> ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
    ```
* For all tables
  
    Use the following command to enable identification key logging for the entire database:
    ```sql
    ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
    ```

**To enable full supplemental logging**

You can enable full supplemental logging for individual tables or all tables in the database:
* For individual tables

    Use the following commands to enable minimal supplemental logging for the database, and then enable full supplemental logging for each table that you want to use:
    ```sql
    ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
    ALTER TABLE <schema name>.<table name> ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ```
* For all tables

    Use the following command to enable full supplemental logging for the entire database:
    ```sql
    ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
    ```
3. To submit the changes:
    ```sql
    ALTER SYSTEM SWITCH LOGFILE;
    ```
## Task 3. Create a User Account
Create a user account to use with the Oracle CDC Client origin. You need the account to access the database through JDBC.

You create accounts differently based on the Oracle version that you use:
<!-- 
**Oracle 12c, 18c, or 19c multitenant databases**

For multitenant Oracle 12c, 18c, or 19c databases, create a common user account. Common user accounts are created in cdb$root and must use the convention: ```c##<name>```.
1. Log into the database as a user with DBA privileges.
2. Create the common user account.

    Use the following set of commands for Oracle 12c or 18c;
    ```sql
    ALTER SESSION SET CONTAINER=cdb$root;
    CREATE USER <user name> IDENTIFIED BY <password> CONTAINER=all;
    GRANT create session, alter session, set container, logmining, execute_catalog_role TO <user name> CONTAINER=all;
    GRANT select on GV_$DATABASE to <user name>;
    GRANT select on V_$LOGMNR_CONTENTS to <user name>;
    GRANT select on GV_$ARCHIVED_LOG to <user name>;
    ALTER SESSION SET CONTAINER=<pdb>;
    GRANT select on <db>.<table> TO <user name>;
    ```
    Repeat the final command for each table that you want to use.

    Use the following set of commands for Oracle 19c:
    ```sql
    ALTER SESSION SET CONTAINER=cdb$root;
    CREATE USER <user name> IDENTIFIED BY <password> CONTAINER=all;
    GRANT create session, alter session, set container, logmining, execute_catalog_role TO <user name> CONTAINER=all;
    GRANT select on GV_$DATABASE to <user name>;
    GRANT select on V_$LOGMNR_CONTENTS to <user name>;
    GRANT select on GV_$ARCHIVED_LOG to <user name>;
    GRANT select on V_$LOG to <user name>;
    GRANT select on V_$LOGFILE to <user name>;
    GRANT select on V_$LOGMNR_LOGS to <user name>;
    ALTER SESSION SET CONTAINER=<pdb>;
    GRANT select on <db>.<table> TO <user name>;
    ```
    Repeat the final command for each table that you want to use.

When you configure the origin, use this user account for the JDBC credentials. Use the entire user name, including the `c##`, as the JDBC user name.

**Oracle 12c, 18c, or 19c standard databases**

For standard Oracle 12c, 18c, or 19c databases, create a user account with the necessary privileges:
1. Log into the database as a user with DBA privileges.
2. Create the user account.
Use the following set of commands for 12c or 18c:
```sql
CREATE USER <user name> IDENTIFIED BY <password>;
GRANT create session, alter session, logmining, execute_catalog_role TO <user name>;
GRANT select on GV_$DATABASE to <user name>;
GRANT select on V_$LOGMNR_CONTENTS to <user name>;
GRANT select on GV_$ARCHIVED_LOG to <user name>;
GRANT select on <db>.<table> TO <user name>;
```
Repeat the final command for each table that you want to use.

Use the following set of commands for 19c:
```sql
CREATE USER <user name> IDENTIFIED BY <password>;
GRANT create session, alter session, logmining, execute_catalog_role TO <user name>;
GRANT select on GV_$DATABASE to <user name>;
GRANT select on V_$LOGMNR_CONTENTS to <user name>;
GRANT select on GV_$ARCHIVED_LOG to <user name>;
GRANT select on V_$LOG to <user name>;
GRANT select on V_$LOGFILE to <user name>;
GRANT select on V_$LOGMNR_LOGS to <user name>;
GRANT select on <db>.<table> TO <user name>;
```
Repeat the final command for each table that you want to use.

When you configure the origin, use this user account for the JDBC credentials. -->

**Oracle 11g databases**

For Oracle 11g databases, create a user account with the necessary privileges:
1. Log into the database as a user with DBA privileges.
2. Create the user account:
```sql
CREATE USER <user name> IDENTIFIED BY <password>;
GRANT create session, alter session, execute_catalog_role, select any transaction, select any table to <user name>;
GRANT select on GV_$DATABASE to <user name>;
GRANT select on GV_$ARCHIVED_LOG to <user name>;
GRANT select on V_$LOGMNR_CONTENTS to <user name>;
GRANT select on <db>.<table> TO <user name>;
```
Repeat the final command for each table that you want to use.

When you configure the origin, use this user account for the JDBC credentials.

## Task 4. Extract a Log Miner Dictionary (Redo Logs)
When using redo logs as the dictionary source, you must extract the Log Miner dictionary to the redo logs before you start the pipeline. Repeat this step periodically to ensure that the redo logs that contain the dictionary are still available.

Oracle recommends that you extract the dictionary only at off-peak hours since the extraction can consume database resources.

To extract the dictionary for all standard databases, run the following command:
```sql
EXECUTE DBMS_LOGMNR_D.BUILD(OPTIONS=> DBMS_LOGMNR_D.STORE_IN_REDO_LOGS);
```
To extract the dictionary for Oracle 12c, 18c, or 19c multitenant databases, run the following commands:
```sql
ALTER SESSION SET CONTAINER=cdb$root;
EXECUTE DBMS_LOGMNR_D.BUILD(OPTIONS=> DBMS_LOGMNR_D.STORE_IN_REDO_LOGS);
```
## Task 5. Install the Driver
The Oracle CDC Client origin connects to Oracle through JDBC. You cannot access the database until you install the required driver.

> **Note**: StreamSets has tested the origin with Oracle 11g and 19c with the Oracle 11.2.0 JDBC driver.


# Setup Streamsets for Oracle CDC in a VM in Azure
## Pre-Requisite 1
Open a terminal and set your file descriptors limit to at least 32768
* Step 1 (ulimit):  open the sysctl.conf  and add this line  fs.file-max = 65536

		vi /etc/sysctl.conf   
	add following at end of file in above file:
 
		fs.file-max = 65536
	
	save and exit.

* Step 2 (ulimit):
	
		vi /etc/security/limits.conf
 
	Add following 4 lines in above file  

		*  soft nproc  65535
		*  hard nproc  65535
		*  soft nofile 65535
		*  hard nofile 65535

	save and exit check max open file ulimit 

	Example command to verify and sample output:
	```sh
		ulimit -a
	```
	Increase max user processes in Linux

	* Step 1 (user processes):
	```sh
	vi /etc/security/limits.conf
	```

	add following in above file

			*  soft nproc  65535
			*  hard nproc  65535
			*  soft nofile 65535
			*  hard nofile 65535

	* Step 2 (user processes):
	```sh
	vi /etc/security/limits.d/90-nproc.conf
	```

	add following in above file

  	*  soft nproc  65535
  	*  hard nproc  65535
  	*  soft nofile 65535
  	*  hard nofile 65535
			
	save and exit check the user max processes ulimit 
	```sh
	[root@localhost# ulimit -a
	core file size          (blocks, -c) 0
	data seg size           (kbytes, -d) unlimited
	scheduling priority             (-e) 0
	file size               (blocks, -f) unlimited
	pending signals                 (-i) 127358
	max locked memory       (kbytes, -l) 64
	max memory size         (kbytes, -m) unlimited
	open files                      (-n) 65535
	pipe size            (512 bytes, -p) 8
	POSIX message queues     (bytes, -q) 819200
	real-time priority              (-r) 0
	stack size              (kbytes, -s) 10240
	cpu time               (seconds, -t) unlimited
	**MAX USER PROCESSES              (-U) 65535**
	virtual memory          (kbytes, -v) unlimited
	file locks                      (-x) unlimited
	```
## Pre-Requisite 2
Download and install OpenJDK 8 or Java 8 JDK. (You must have Java 8 JDK, not Java 8 JRE.) 
```sh
yum install java-1.8.0-openjdk
```

## Install Streamsets
* Link: https://streamsets.com/documentation/datacollector/latest/help/datacollector/UserGuide/Installation/FullInstall_ServiceStart.html#concept_e45_3dr_bx
* Link RPM: https://streamsets.com/products/dataops-platform/open-source/
* Link (latest version - have to login): https://accounts.streamsets.com/archives
```sh
# Download RPM Full
wget https://archives.streamsets.com/datacollector/3.16.1/rpm/el7/activation/streamsets-datacollector-3.16.1-el7-activation-all-rpms.tar

# Extract tar
tar xf stream*
# Install
cd streamsets-datacollector-3.16.1-el7-activation-all-rpms
sudo yum localinstall streamsets*.rpm
# Start
sudo systemctl start sdc
# Startup
sudo systemctl enable sdc
```
Link: http://localhost:18630

## Libraries
```sh
sudo -i
# External Library using Package Manager (ui)
sudo chown -R sdc:sdc /opt/streamsets-datacollector/streamsets-libs
# Add the following lines to change permissions
sudo nano /etc/sdc/sdc-security.policy

// user-defined external directory
grant codebase "file:///opt/streamsets-datacollector/streamsets-libs/-" {
  permission java.security.AllPermission;
};

systemctl restart sdc


# Some variables if security policy error
# export SDC_CONF=$SDC_DIST/etc/sdc
# export SDC_DATA=$SDC_DIST/var/lib/sdc
# export SDC_LOG=$SDC_DIST/var/log/sdc
# export SDC_HOME=$SDC_DIST/opt/streamsets-datacollector

# streamsets dc -verbose
```

# Uninstall Streamsets in Linux
```sh
systemctl stop sdc
yum erase streamsets-datacollector
rm -rf /etc/sdc /var/log/sdc /var/lib/sdc /var/lib/sdc-resources
rm -rf /opt/streamsets-datacollector
```

## Oracle JDBC drivers
Version used in OCI: Database Release: 11.2.0.4 \
Link: https://www.oracle.com/database/technologies/jdbcdriver-ucp-downloads.html
```sh
sudo -i
# Download Oracle JDBC driver (rpm)
cd /tmp
# Select the relevant version (ojdbc6.jar) worked for Oracle Database 11.2.0.4
# downlaod ojdbc6-full.tar.gz
wget https://download.oracle.com/otn/utilities_drivers/jdbc/11204/ojdbc6.jar
# gunzip < ojdbc8-full.tar.gz | tar xvf -
#sudo cp OJDBC8-full/*.jar /opt/streamsets-datacollector/streamsets-libs/streamsets-datacollector-jdbc-lib/lib/
sudo ojdbc6.jar /opt/streamsets-datacollector/streamsets-libs/streamsets-datacollector-jdbc-lib/lib/
sudo chown -R sdc:sdc /opt/streamsets-datacollector/streamsets-libs/streamsets-datacollector-jdbc-lib/lib/ojdbc6.jar
sudo systemctl restart sdc
```
## Install on Docker (not tested)
## Adding Custom External Stage Libraries
* https://archives.streamsets.com/datacollector/latest/tarball/enterprise/index.html
* https://streamsets.com/documentation/datacollector/latest/help/datacollector/UserGuide/Configuration/CustomStageLibraries.html#concept_pmc_jk1_1x

```sh
# Install docker container
docker run --restart on-failure -p 18630:18630 -d --name streamsets -v /home/aceso/streamsets/streamsets-datacollector-user-libs:/opt/streamsets-datacollector-user-libs/ streamsets/datacollector

# Within the streamsets cotainer run:
cd /opt/streamsets-datacollector-user-libs/
or
cd $USER_LIBRARIES_DIR
```

# Appendix
## LogMiner Dictionary Source
LogMiner provides dictionaries to help process redo logs. LogMiner can store dictionaries in several locations.

The Oracle CDC Client can use the following dictionary source locations:
Online catalog - Use the online catalog when table structures are not expected to change.
Redo logs - Use redo logs when table structures are expected to change. When reading the dictionary from redo logs, the Oracle CDC Client origin determines when schema changes occur and refreshes the schema that it uses to create records. The origin can also generate events for each DDL it reads in the redo logs.
Important: When using the dictionary in redo logs, make sure to extract the latest dictionary to the redo logs each time table structures change. For more information, see Task 4. Extract a Log Miner Dictionary (Redo Logs).
Note that using the dictionary in redo logs can have significantly higher latency than using the dictionary in the online catalog. But using the online catalog does not allow for schema changes.

For more information about dictionary options and configuring LogMiner, see the Oracle LogMiner documentation.


## Commands in Oracle
```sql
create schema WITS

# Get database name (should return: GLSAGABO)
select * from global_name;

# Get Privilages of user
SELECT * FROM USER_SYS_PRIVS; 
SELECT * FROM USER_TAB_PRIVS;
SELECT * FROM USER_ROLE_PRIVS;

#### Validation Queries for AS400 #####

update smypdta.mamam1 set m1ref2 ='R'      
where m1aldt>'1200701'

update smypdta.fps010 set aeref2 ='R'  
where aldt10>'1200701'

select * from fps010  
where aldt10>'1200701'

# Get timestamp from database server
Select sysdate, timestamp_to_scn(sysdate) from dual;

Select CURRENT_SCN from v$database;

```

# Streamsets Oracle CDC issues
## Missing log file error starting Logminer
Must Restart Orgin and Strat pipeline from GUI

Link: https://streamsets.com/blog/replicating-oracle-to-mysql-and-json/#ora-01291-missing-logfile

## SQL Mapping in Streamsets
```sql
Tables: ${record:attribute('jdbc.tables')}
```

# Streamsets Configurations
## CASA Tables
```json
[
	{
		"schema": "NAGENCY2",
		"table": "T_BOOK_HIST_TRACK"
	},
	{
		"schema": "NAGENCY2",
		"table": "T_BOOKING"
	},
	{
		"table": "T_CAC_BOOKED",
		"schema": "NAGENCY2"
	},
	{
		"table": "T_CARGO_BOOKED",
		"schema": "NAGENCY2"
	},
	{
		"schema": "NAGENCY2",
		"table": "T_CASA_BOOKING_OFFICE"
	},
	{
		"schema": "NAGENCY2",
		"table": "T_CASA_CUSTOMER"
	},
	{
		"schema": "NAGENCY2",
		"table": "T_VOYAGE_ALLOCATION_DETAIL"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_CAC_TYPE"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_CODA_SEGMENT_CARGO_CLASS"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_COMMODITY_PARTIC_DETAIL"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_PARTY"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_REGIONAL_GLOBAL_DETAIL"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_REGIONAL_GLOBAL_PARTY"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_PORT"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_STOW_TON"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_TRADE_ROUTE"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_VESSEL_PARTICULARS"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_VESSEL_TYPE_CAPACITY"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_VOYAGE_DETAIL"
	},
	{
		"schema": "NAGENCY2",
		"table": "TC_VOYAGE_HEADER"
	}
]

```

# Migration Tools
Link: http://www.sqlines.com/online