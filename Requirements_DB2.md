Created by: J.Murray\
Date: 1st July 2020
1. - [x] Firewalls configured in Azure 
2. - [x] VMs in Azure with access to OCI and DB2/AS400 (DONE)
3. **CDC Options**
   1. - [X] Syniti
   2. - [X] Talend Data Integration
   3. - [x] GoldenGate
   4. - [ ] Streamsets (Opensource)
# Requirements for DB2 for i
# GoldenGate
## Instructions for **IBM-DB2** for i source
Link: https://docs.oracle.com/goldengate/c1230/gg-winux/GGHDB/using-oracle-goldengate-db2-i.htm#GGHDB-GUID-F382412C-640E-473F-8223-37B0A7AB04C6

With Oracle GoldenGate for DB2 for i, you can:

* Map, filter, and transform transactional data changes between similar or dissimilar supported DB2 for i versions, and between supported DB2 for i versions and other supported types of databases.

* Perform initial loads from DB2 for i to target tables in DB2 for i or other databases to instantiate a synchronized replication environment.

## Support List Matrix for DB2 for i
### Data Types
![](GoldenGate/images/gg1.png)
### Objects, OPerations and Parameters
![](GoldenGate/images/gg2.png)

## Preparing the System for Oracle GoldenGate
Summary list of Item requiring access:
1. **Journals Enabled** for Data Capture by Extract
   1. **Allocating** Journals to an **Extract Group**
   2. Setting **Journal Parameters**
   3. **Deleting** Old Journal Receivers   
2. Specifying **Object Names**
   1. Assigning Row Identifiers
   2. Preventing Key Changes
   3. Disabling Constraints on the Target
   4. Enabling Change Capture
   5. Maintaining Materialized Query Tables
   6. Specifying the Oracle GoldenGate Library
3. Adjusting System Clock
4. Configuring the ODBC Driver

# Syniti
When setting up IBM DB2 for i for use with Syniti Data Replication:

1. **Set user ID authorities** appropriately (DBMS tools)
2. Make sure tables are **journaled** and **receivers** are set up appropriately (DBMS tools)
3. **Create a library** on Db2 (Management Center)
   
When replicating from IBM Db2 for i using mirroring or synchronization, three transactional modes are available: 
1. Log Reader
2. Log Server Agent (**Preferred according to table below**)
3. Log Reader API
   > **Note**: The Log Reader API is required when replicating tables with LOB values. Log Reader API option is available for Db2 for i V6R1 and above

![](Syniti\images\log_type.png)

In the Wizard\
![](Syniti\images\image2.png)


For all three modes, you first need to:
### **Setting up user with authorities**
Your user ID needs the authority to run the following commands:

* DSPOBJD on the schema used for replication

and to access the following system tables:

* QSYS2.SYSTABLES
* QSYS2.SYSCOLUMNS
* QSYS2.SYSCST
* QSYS2.SYSKEYCST
* QSYS2.SYSKEYS
* QSYS2.SYSINDEXES
* QSYS2.SYSPARTITIONSTAT
* SYSIBM.SQLFOREIGNKEYS*

Additionally, only if the system is used as source in mirroring or synchronization mode, the user ID needs access to:

* DSPJRN (on the journals/receivers involved in replication)
* DSPFD (on the tables involved in replication)
* DSPFFD (on the tables involved in replication)
 
For synchronization, the user ID should be used exclusively for Syniti Data Replication so that transactions executed by Syniti DR can be easily identified in the journal.
### **CREATING THE LIBRARY on DB2 for i**
**Set up a library on your Db2 system**. This step sets up the library by `transferring` a `savf` file (I think via `FTP`) to the Db2 server and creating the DBMOTOLIB library (or a library name of your choosing). You can also perform this procedure *manually* for more complete control over operations or in case the automatic process does not work.
* **The library name**
  * DBMOTOLIB, is supplied. This field can be modified to supply a different library name. This is the name of the library that will be created on the Db2 system. It can be useful to change the default library name when, for example, you have two CDC installations using the same database server, and you wish to keep separate libraries for each installation.
* **Savf File**
  * In iSeries(AS400) Save files are very handy in saving objects, complete libraries, IFS directories and files and then restoring them back on the same or different machine. You can even save a save file inside another save file.
* **FTP access**
  * A user ID with write permissions and `QSECOFR` privileges on the Db2 server

#### **Creating the Db2 Library Manually Steps - Create Stored Procedure**
If you are unable to create the library for your Db2 system automatically, you can access the appropriate savefile in the ServerFiles folder and restore it manually as described below. Use the name of the restored library when configuring your System i connection in the Management Center. The default name provided in the Management Center is DBMOTOLIB, so either use this name in the instructions below, or be sure to change the name in the Management Center.

If more than one Syniti Data Replicationinstallation is sharing the same IBM i server, be sure to set up a different library for each installation.

Note that operating system version V3R2 or higher is required for Syniti Data Replication 6.0.0 or higher.

1. Create a temporary folder on your PC (e.g., C:\DBMLib)
2. Copy the appropriate savefile for your i operating system version from the ServerFiles folder to C:\DBMLib. The savefiles are: V6R1 and above. For use with Log Reader API transaction mode: `DBMLIBAPI61.SAVF`
3. Run the DOS command prompt and change the working directory to C:\DBMLib.
   ```
   C:>cd C:\DBMLib
   ```
4. Run an FTP session followed by the Db2 system IP address
   ```
   C:\DBMLib> ftp nj3m1b.wwl.2wglobal.com
   ```
5. Insert your username and password when prompted. Make sure that your user ID has write permissions and QSECOFR privileges.

6. Make `QGPL` the current directory.
   ```
   ftp> quote cwd QGPL
   ```
7. Create an `empty Savefile` on the Db2 system.
   ```
   ftp> quote rcmd CRTSAVF FILE(QGPL/DBMLIBSAVF) AUT(*ALL)
   ```
8. Switch to `BINARY` mode.
   ```
   ftp> bin
   ```
9. `Transfer` the savefile. In the command below, replace DBMLIB.SAVF with the name of the savefile you are using.
   ```
   ftp>  put DBMLIB.SAVF DBMLIBSAVF
   ```
10. `Restore` the savefile, for example in a library called MYDBMOTOLIB:
      ```
      ftp> quote rcmd RSTLIB SAVLIB(DBMOTOLIB) DEV(*SAVF) SAVF(QGPL/DBMLIBSAVF) MBROPT(*ALL) 
      ALWOBJDIF(*ALL) RSTLIB(MYDBMOTOLIB)
      ```
      > Note: If using the  default library name, DBMOTOLIB, be sure to replace only the library name in the command RSTLIB(MYDBMOTOLIB). SAVLIB(DBMOTOLIB)indicates the name of the library as saved in the SAVF file. The RSTLIB parameter instead indicates the name of the library where you want to restore the SAVF file which by default is the name of the library saved in the SAVF.

11.  Delete the save file.
      ```
      ftp>  quote dele DBMLIBSAVF
      ```
12.   Close the ftp session
      ```
      ftp> quit
      ```
13.  Finally, create the stored procedure `DBMOTOLIB.JRNSQNM` on the Db2 system. This needs to be executed as a SQL command.
      > Note: If using the IBM i console to perform this operation, use "/" instead of "." below in `"DBMOTOLIB.JRNSQNM"` to give you `"DBMOTOLIB/JRNSQNM"`


Operating System Version V5R4 or greater:
Use with `DBMLIB54.SAVF` and the relevant procedure is as shown below:
```sql
CREATE PROCEDURE DBMOTOLIB.JRNSQNM
(IN JOUR CHAR(10),
IN JLIB CHAR(10),
IN FNMS CHAR(900),
IN JDAT CHAR(8),
IN JTIM CHAR(6),
IN JCDE CHAR(100),
INOUT NUMSEQ CHAR(20),
INOUT RECVR CHAR(10),
INOUT LIBRCV CHAR(10),
OUT LSTSQN CHAR(20),
OUT LSTTMSP CHAR(26),
OUT LSTRECVR CHAR(10),
OUT LSTLIBREC CHAR(10),
OUT FLAG CHAR(1),
OUT CODC CHAR(7),
OUT MSGG CHAR(100))
LANGUAGE CL SPECIFIC DBMOTOLIB.JRNSQNM
NOT DETERMINISTIC
NO SQL
CALLED ON NULL INPUT
EXTERNAL NAME 'DBMOTOLIB/JRNSQNM'
PARAMETER STYLE GENERAL
```
## IBM i System Journals and Receivers
If you are performing mirroring or synchronization with an Db2 system source table, you need to manage the journals and receivers for your source tables. Typically, your system administrator manages journals and receivers, but it is helpful to know a little about the operations involved.

A journal is a collector of modified data from "journaled" files. The modifications that occurred in the files are detailed and written in a receiver as a log of the operations performed on the physical file.

The terminology is a little misleading: a journal is not, as you would expect, the place where modifications are tracked, but only the reference to write them on a receiver. A receiver is the physical location where traced modifications are written.

When a journal is started, it must be associated with a receiver. Receivers are set to a defined size that can be configured to fit your needs. The receiver is "bound" to the journal.

A physical file cannot be associated with more than one journal at the same time, like a journal cannot be associated with more than one receiver at a time. However, the same journal can track information for many physical files. If you want to change the current journal/receiver setting for a file, you can stop the journaling for that file and associate the file with a different one, but not at the same time.

Logical files (view, indexes, etc.), as well as LOB data types, are not journaled. Because a file must be journaled to be replicated in mirroring or synchronization modes, logical files can only be replicated in refresh mode.

> I believe the following are recommendations only - lets see if it recognises with us not follwoing this?? - JM

#### Journal and Receiver Names
Use receiver names in which the last 4 or 5 character of the name are digits, starting with 00001, like  

    DBRSJ00001

Then create a new receiver with a command like

    CRTJRNRCV JRNRCV(DBRSWRK/DBRSJ00001) AUT(*ALL)

and the journal with the command

    CRTJRN JRN(DBRSWRK/DBRSJ) JRNRCV(DBRSWRK/DBRSJ00001) AUT(*ALL)

Note the names of journal and receiver: the second the second is the same as the first with a "00001" suffix.

It is possible to create and use journals with minimized entries:

CRTJRN JRN(DBRSWRK/DBRSJ) JRNRCV(DBRSWRK/DBRSJ00001) AUT(*ALL)MINENTDTA(*FLDBDY)

 Adding the MINENTDTA(*FLDBDY) option to the CRTJRN command will decrease the size of journal entries as follows. The Minimized Entry Specific Data (MINENTDTA) parameter for an object type allows entries for that object type, in this case a database physical file, to be minimized. While *FILE, *DTAARA, and *FLDBDY values are allowed the MINENTDTA parameter in the CRTJRN command, Syniti DRsupports only *FLDBDY (field boundaries.) This means that the minimizing will occur on field boundaries. The entry specific data will be viewable and may be used for auditing purposes.

A journal configured with MINENTDTA(*FLDBDY) only saves values for changed columns: changes to primary keys are not logged in the journal. To force one or more columns to be saved in the journal even if not changed, create a journaled index on those columns. Syniti DR requires primary key values to be present for each change, therefore it is necessary to create a journaled index on all the fields mapped to a target key.

Below is an example of how to set a table that Syniti DR can replicate. For the following table and index, assuming that HITTEST2/QSQJRN is the journal name:

      create table hittest2.test1 (id integer, name char(20),address varchar(50), dbo date)

      create index hittest2.test1idx on hittest2.test1 (id)

you should journal the index to ensure that the ID values appear in the journal:

      STRJRNAP FILE(HITTEST2/TEST1) JRN(HITTEST2/QSQJRN)

Finally, use the command

      CHGJRN JRN(DBRSWRK/DBRSJ) JRNRCV(*GEN)

The option (*GEN) enables automatic naming for receivers based on last number + 1.

If you need to have "different" names for journal and receivers, you can set the automatic naming management using the following command:

      CHGJRN JRN(DBRSWRK/DBRSJ) MNGRCV(*SYSTEM)

(which is automatically set when using the JRNRCV(*GEN) option)

Using  automatic naming management,  MNGRCV(*USER), for receivers is useful because when the current receiver ("attached" receiver) becomes full, the system will automatically unbind the attached receiver, then create and bind a new receiver with a name based on last number + 1, i.e.

      DBRSJ00001 --> DBRSJ00002

#### Activating a Journal
STRJRNPF FILE(DBRSWRK/TableFILE) JRN(DBRSWRK/DBRSJ) IMAGES(*BOTH) OMTJRNE(*OPNCLO)

The IMAGES parameter can be set to either *BOTH or *AFTER. It is recommended that you set the IMAGES parameter to *BOTH. This option saves the record’s image in the log, before and after the update command, and is requested by Syniti DR in order to correctly manage the record’s primary key. If you choose to set the IMAGES parameter to *AFTER, you will need to modify the target table by adding a Relative Record Number (RRN) field and mapping it to the source table !RecordID field as follows:

1. Create a Decimal(15) field on the target table and make it the primary key.
2. Create source and target connections for the replication.
3. Define the replication.
4. When mapping source and target fields, right click on the target table field you want to map the RRN to, and choose Map to Expression...
5. In the upper pane of the Expression Editor, type [!RecordID].
Alternatively, you can expand Values in the lower left hand pane, click on Log Field, then double click on !RecordID in the lower right hand pane.
6. Click `OK` to save the expression.
   
> Notes:

* You should not set the IMAGES parameter to *AFTER if you are planning to perform synchronization (bi-directional mirroring).  
* If the source table is reorganized, you need to run a refresh replication on the target table (to update RRN changes) before mirroring can proceed again.
* Syniti DR can report member reorganization in the log:
In the Log Settings dialog, check the option "Write a warning on member reorganize" to record a warning in the log if RGZPFM (reorganized physical member) is executed on a table.
#### Receiver Management
When a receiver is unbound for any reason (because it becomes full, manual management, system management), you can choose what to do with it between:     

* Unbind it and let it remain on disk

* Unbind and delete it

This assumes that you have automated the receiver management as described before, and therefore excludes a manual intervention: you are giving the system the appropriate instructions in order to manage receivers by itself.

In order to allow the receiver to be unbound but not deleted, you need to run the following command on your journal object:     

     CHGJRN JRN(DBRSWRK/DBRSJ) DLTRCV(*NO)

The default is *YES. Using this setting, the system will unbind old receivers and keep them available on disk. The unbound receivers available on disk are called "online" (their status), while the unique and currently bound receiver is called "attached".

If you need to change the system management to delete old receivers when they're unbound, you need to run

     CHGJRN JRN(DBRSWRK/DBRSJ) DLTRCV(*YES)

#### Sequence Number for Journal Entries
A receiver is the final container which keeps track of every operation (aka transaction) performed on the physical files with which they are associated via the journal. Every operation tracked is completely described and numbered with a list of details - date, time, kind of operation, etc. - and a numeric progressive value, named  "Sequence number".

You can choose to set the sequence number for journal entries to continue among several receivers or to be reset at every receiver change. Run the command

    CHGJRN JRN(xxx) JRNRCV(*GEN) SEQOPT(*CONT)

to allow your journal entry sequence number to continue in the subsequent receivers when the attached receiver is unbound.

Normally, when you change journal receivers, you continue the sequence number for journal entries. When the sequence number becomes very large, you should consider resetting the sequence to start the numbering at 1.

You can reset the sequence number only when all changes are forced to auxiliary storage for all journaled objects and commitment control is not active for the journal. Resetting the sequence number has no effect on how the new journal receiver is named.

If you use system journal-receiver management for a journal, the sequence number for the journal is reset to 1 whenever you restart the system or vary on the independent disk pool containing the journal. When you restart the system or vary on an independent disk pool, the system performs the change journal operation for every journal on the system or disk pool that specifies system journal-receiver management.

The operation that the system performs is equivalent to

    CHGJRN JRN(xxx) JRNRCV(*GEN) SEQOPT(*RESET)

The sequence number is not reset if journal entries exist that are needed for commitment control IPL recovery.                                          

The maximum sequence number is 2147483136. If you specified RCVSIZOPT(*MAXOPT1) or RCVSIZOPT(*MAXOPT2) for the journal that you attached the receiver to, then the maximum sequence number is 9999999999. If this number is reached, journaling stops for that journal.

When the size limit is reached (or a manual intervention occurs), a system-managed receiver is unbound. If the delete option is disabled, it becomes an "online" receiver, meaning it is not attached but still available.

#### Sequence Numbers and Mirroring
Syniti DR keeps all the information about the current status of the replication, including the journal entry number and corresponding receiver, in the metadata.

It is important to have the sequence number for journal entries to continue from receiver to receiver because Syniti DR, when replicating in mirroring mode from the i/iSeries/AS400, performs comparisons between the last used (mirrored) journal entry and the current one. If the replicator detects a current value lower than the last managed value (stored in the metadata), it stops replicating and reports a message saying that a potential problem has occurred.

For this reason, it is important to keep old receivers on disk without deleting them (keep them "online".) Syniti DR stores journal entry numbers so that if a change receiver (or more than one) occurs and the last managed entry is now in an unbound receiver but the receiver is still online, the data are retrieved and the mirroring process continue without problems. If, instead, the last managed journal entry is in a deleted receiver, a potential problem is signalled in the log because something unexpected has occurred, and transactions could be missing. Syniti DR also stops replicating the source files associated with that journal,

The information stored in an online receiver is still retrievable using simple i/iSeries/AS400 CL commands. Syniti DR uses these commands. Deleted receivers cannot be used at all, so if they are not yet processed, the information they hold is lost – and, when mirroring, even a potential loss of information is a problem.

Even if a new receiver is created and bound, it does not help. If, for instance, the Replication Agent was stopped BEFORE it had processed all the journal entries in the old receiver, data may already be (potentially) missing. As a general rule, it is appropriate for the Replication Agent to signal an error and stop.

A contextual stop for Syniti DR when an IPL is performed or backup operations are in place on the i/iSeries/AS400 is a very good rule. Journaling and backup at the same time can often lead to problems. 

## Synchronization Limitation with IBM Db2 for i
If you are planning a synchronization replication with IBM Db2 for i (iSeries/AS400) as a source or target database, note that any CLRPFM commands which Syniti Data Replication encounters in the journal will not be propagated to the destination table. Synchronization does not support the use of clear on physical members, although clear commands are supported for mirroring replications.

To work around this restriction, avoid calling the CLRPFM command, replacing it with DELETE FROM TABLE without adding a WHERE condition.

# Talend ESB Studio
Link: https://help.talend.com/reader/RLLVjeyBol_aw8PB0tyxJg/OP5L_4_6MOlroVa0kFLE0Q

Prior to setting up CDC in Redo/Archive log mode (journal) on AS/400, you need to verify the prerequisites as follows on your AS/400:

the AS/400 user account for CDC must have *ALLOBJ privileges or at least all of the following **privileges**:

- CRTSAVF
- CLRSAVF
- DLTF
- RSTLIB
- DLTLIB
- CRTLIB
- CHGCMD
- FTP (access to the FTP port must be ensured),
- READ access on journal receivers
- READ access on monitored AS/400 files
- READ/WRITE access on output library

**File names:**
* The names of the files of interest should not exceed 10 characters;

* If the files of interest are already journalized, the journal must be created with option IMAGES (*BOTH)
