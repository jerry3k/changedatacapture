# CDC- Oracle GoldenGate

# Instructions for **IBM-DB2** for i source
Link: https://docs.oracle.com/goldengate/c1230/gg-winux/GGHDB/using-oracle-goldengate-db2-i.htm#GGHDB-GUID-F382412C-640E-473F-8223-37B0A7AB04C6

With Oracle GoldenGate for DB2 for i, you can:

* Map, filter, and transform transactional data changes between similar or dissimilar supported DB2 for i versions, and between supported DB2 for i versions and other supported types of databases.

* Perform initial loads from DB2 for i to target tables in DB2 for i or other databases to instantiate a synchronized replication environment.

## Support List Matrix for DB2 for i
### Data Types
![](images/gg1.png)
### Objects, OPerations and Parameters
![](images/gg2.png)

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

# Instructions for **Oracle** source
Follow here: https://git.aceso.no/jerry3k/cdc-walwil/src/branch/master/Streamsets/CDC-REQS.md#user-content-oracle-cdc

