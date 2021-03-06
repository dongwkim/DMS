This document describes how to build datalake on S3 from multiple data sources and ETL with glue

# Create S3 bucket

**Bucket Name** : `oracle-to-s3-dms-kiwony`

<kbd> ![GitHub Logo](images/1.png) </kbd>

# Create IAM Policy & Role

**Policy** : `prod.dms.s3.access.policy`

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::oracle-to-s3-dms-kiwony*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::oracle-to-s3-dms-kiwony*"
            ]
        }
    ]
}
```

### Create Role

**Role** : `prod.dms.s3.access.role`
**Attach Policy** : `prod.dms.s3.access.policy`

**Select "DMS" as use case**

<kbd> ![GitHub Logo](images/2.png) </kbd>

**Attach the created policy**

<kbd> ![GitHub Logo](images/3.png) </kbd>

**Create Role**

<kbd> ![GitHub Logo](images/4.png) </kbd>

# Create DB Instance

**AMI-ID** : `ami-0e63e3e96d7d9e456`

<kbd> ![GitHub Logo](images/5.png) </kbd>

<kbd> ![GitHub Logo](images/6.png) </kbd>

<kbd> ![GitHub Logo](images/7.png) </kbd>

<kbd> ![GitHub Logo](images/8.png) </kbd>

<kbd> ![GitHub Logo](images/9.png) </kbd>

<kbd> ![GitHub Logo](images/10.png) </kbd>

<kbd> ![GitHub Logo](images/11.png) </kbd>

# Connect to Oracle Instance console

<kbd> ![GitHub Logo](images/12.png) </kbd>

### Connect -> Session Manager

<kbd> ![GitHub Logo](images/13.png) </kbd>

### Execute following commands on session manager console

```
sudo su -
su - oracle
./startup.sh
```

<kbd> ![GitHub Logo](images/14.png) </kbd>

# DMS Pre-requirement for Oracle

### Execute following commands on session manager console

```
oracle@oracle11g:/home/oracle> sqlplus / as sysdba

create user dms_user identified by <PASSWORD> default tablespace users temporary tablespace temp quota unlimited on users;
grant connect, resource to dms_user;
grant EXECUTE ON dbms_logmnr to dms_user;

GRANT SELECT ANY TRANSACTION to dms_user;
GRANT SELECT on V_$ARCHIVED_LOG to dms_user;
GRANT SELECT on V_$LOG to dms_user;
GRANT SELECT on V_$LOGFILE to dms_user;
GRANT SELECT on V_$DATABASE to dms_user;
GRANT SELECT on V_$THREAD to dms_user;
GRANT SELECT on V_$PARAMETER to dms_user;
GRANT SELECT on V_$NLS_PARAMETERS to dms_user;
GRANT SELECT on V_$TIMEZONE_NAMES to dms_user;
GRANT SELECT on V_$TRANSACTION to dms_user;
GRANT SELECT on ALL_INDEXES to dms_user;
GRANT SELECT on ALL_OBJECTS to dms_user;
GRANT SELECT on DBA_OBJECTS to dms_user;
GRANT SELECT on ALL_TABLES to dms_user;
GRANT SELECT on ALL_USERS to dms_user;
GRANT SELECT on ALL_CATALOG to dms_user;
GRANT SELECT on ALL_CONSTRAINTS to dms_user;
GRANT SELECT on ALL_CONS_COLUMNS to dms_user;
GRANT SELECT on ALL_TAB_COLS to dms_user;
GRANT SELECT on ALL_IND_COLUMNS to dms_user;
GRANT SELECT on ALL_LOG_GROUPS to dms_user;
GRANT SELECT on SYS.DBA_REGISTRY to dms_user;
GRANT SELECT on SYS.OBJ$ to dms_user;
GRANT SELECT on DBA_TABLESPACES to dms_user;
GRANT SELECT on ALL_TAB_PARTITIONS to dms_user;
GRANT SELECT on ALL_ENCRYPTED_COLUMNS to dms_user;
GRANT SELECT on V_$LOGMNR_LOGS to dms_user;
GRANT SELECT on V_$LOGMNR_CONTENTS to dms_user;
GRANT SELECT on ALL_VIEWS to dms_user;
GRANT SELECT ANY TABLE to dms_user;
GRANT ALTER ANY TABLE to dms_user;
GRANT create any directory to dms_user;
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
ALTER TABLE OSHOP.DUMMY ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
ALTER TABLE OSHOP.EMP ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
ALTER TABLE OSHOP.DEPT ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
ALTER TABLE OSHOP.BIGEMP ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
ALTER TABLE OSHOP.BONUS ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
ALTER TABLE OSHOP.SALGRADE ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;

```

# Database Migration Service

## Create Replication Instance

<kbd> ![GitHub Logo](images/15.png) </kbd>

## Create Source Endpoint(Oracle)

<kbd> ![GitHub Logo](images/16.png) </kbd>

<kbd> ![GitHub Logo](images/17.png) </kbd>

<kbd> ![GitHub Logo](images/18.png) </kbd>

<kbd> ![GitHub Logo](images/19.png) </kbd>

### Extra Connections attributes : includeOpForFullLoad=true;cdcInsertsOnly=true

**This endpoint will capture insert operation only**

<kbd> ![GitHub Logo](images/20.png) </kbd>

## Create Target Endpoint(S3)

<kbd> ![GitHub Logo](images/21.png) </kbd>

<kbd> ![GitHub Logo](images/22.png) </kbd>

## Create Migration Task

<kbd> ![GitHub Logo](images/23.png) </kbd>

<kbd> ![GitHub Logo](images/24.png) </kbd>

## Check Table Statistics

<kbd> ![GitHub Logo](images/25.png) </kbd>

## Check S3 Bucket

**4 Folders created**

<kbd> ![GitHub Logo](images/26.png) </kbd>

## Check Initial Loading Data in S3 bucket

**4 Folders created**

<kbd> ![GitHub Logo](images/27.png) </kbd>

<kbd> ![GitHub Logo](images/28.png) </kbd>

<kbd> ![GitHub Logo](images/29.png) </kbd>

## Connect to System Manager Console to raise new insert operation

<kbd> ![GitHub Logo](images/30.png) </kbd>

```
sh-4.2$ sudo su -
Last login: Mon Apr 27 08:22:56 EDT 2020 on pts/0
root@oracle11g:/root# su - oracle
Last login: Mon Apr 27 08:22:57 EDT 2020 on pts/0
oracle@oracle11g:/home/oracle> sqlplus oshop/<PASSWORD>

SQL*Plus: Release 11.2.0.4.0 Production on Mon Apr 27 20:17:14 2020

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> update emp set sal=3333 where empno=7777;
1 row updated.

SQL> update emp set comm=2222 where empno=7774;
1 row updated.

SQL> commit;
Commit complete.

SQL> insert into emp values (8888, 'yoon', 'PSA', 7902, sysdate, 4000, 1000, 10);
SQL> commit;
SQL> insert into emp values (8890, 'jenny', 'PSA', 7902, sysdate, 6000, 3000, 10);
SQL> commit;
SQL> insert into emp(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, DEPTNO) values (9999, 'TOM', 'PSA', 7902, sysdate, 5000, 10);
SQL> commit;
```

<kbd> ![GitHub Logo](images/31.png) </kbd>

<kbd> ![GitHub Logo](images/32.png) </kbd>

# Glue & Athena

### Create Database

<kbd> ![GitHub Logo](images/33.png) </kbd>

### Add tables using a crawler

<kbd> ![GitHub Logo](images/34.png) </kbd>

<kbd> ![GitHub Logo](images/35.png) </kbd>

<kbd> ![GitHub Logo](images/36.png) </kbd>

<kbd> ![GitHub Logo](images/37.png) </kbd>

<kbd> ![GitHub Logo](images/38.png) </kbd>

<kbd> ![GitHub Logo](images/39.png) </kbd>

<kbd> ![GitHub Logo](images/40.png) </kbd>

<kbd> ![GitHub Logo](images/41.png) </kbd>

<kbd> ![GitHub Logo](images/42.png) </kbd>

<kbd> ![GitHub Logo](images/43.png) </kbd>

<kbd> ![GitHub Logo](images/44.png) </kbd>

<kbd> ![GitHub Logo](images/45.png) </kbd>

<kbd> ![GitHub Logo](images/46.png) </kbd>

<kbd> ![GitHub Logo](images/47.png) </kbd>

<kbd> ![GitHub Logo](images/48.png) </kbd>

### View Table metadata from Glue Database

<kbd> ![GitHub Logo](images/49.png) </kbd>

### Query Data from Athena

<kbd> ![GitHub Logo](images/50.png) </kbd>

<kbd> ![GitHub Logo](images/51.png) </kbd>

### Create Athena View

<kbd> ![GitHub Logo](images/52.png) </kbd>

<kbd> ![GitHub Logo](images/53.png) </kbd>

dms-vpc-role
AmazonDMSVPCManagementRole

dms-cloudwatch-logs-role
AmazonDMSCloudWatchLogsRole
