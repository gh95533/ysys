[TOC]

# oracle learn 



```
Connected to an idle instance.

SQL> startup nomount;
ORACLE instance started.

Total System Global Area  626327552 bytes
Fixed Size		    2215944 bytes
Variable Size		  473960440 bytes
Database Buffers	  146800640 bytes
Redo Buffers		    3350528 bytes
SQL> desc dual;
ERROR:
ORA-04043: object dual does not exist


SQL> alter database mount;

Database altered.

SQL> desc dual;
ERROR:
ORA-04043: object dual does not exist


SQL> select * from dual;

ADDR		       INDX    INST_ID D
---------------- ---------- ---------- -
0000000009155978	  0	     1 X

SQL> alter database open;
alter database open
*
ERROR at line 1:
ORA-01092: ORACLE instance terminated. Disconnection forced
ORA-00942: table or view does not exist
Process ID: 1824
Session ID: 1 Serial number: 3



```

```
SMON: enabling cache recovery
Successfully onlined Undo Tablespace 2.
Verifying file header compatibility for 11g tablespace encryption..
Verifying 11g file header compatibility for tablespace encryption completed
SMON: enabling tx recovery
Database Characterset is AL32UTF8
No Resource Manager plan active
Errors in file /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_1824.trc:
ORA-00942: table or view does not exist
Errors in file /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_1824.trc:
ORA-00942: table or view does not exist
Error 942 happened during db open, shutting down database
USER (ospid: 1824): terminating the instance due to error 942
Instance terminated by USER, pid = 1824
ORA-1092 signalled during: alter database open...
opiodr aborting process unknown ospid (1824) as a result of ORA-1092
Sun Dec 09 07:13:57 2018
ORA-1092 : opitsk aborting process
```



```
$ cat /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_1824.trc
..
----------------------------------------------
Recovery sets nab of thread 1 seq 53 to 67643 with 8 zeroblks

*** 2018-12-09 07:13:57.035
ORA-00942: table or view does not exist
ORA-00942: table or view does not exist

*** 2018-12-09 07:13:57.045
USER (ospid: 1824): terminating the instance due to error 942
```



```
SQL> select * from dict where table_name ='V$PROCESS';

TABLE_NAME		       COMMENTS
------------------------------ ------------------------------
V$PROCESS		       Synonym for V_$PROCESS

SQL> select * from dict where table_name='DBA_DATA_FILES';

TABLE_NAME		       COMMENTS
------------------------------ ------------------------------
DBA_DATA_FILES		       Information about database dat
			       a files
			      
 SELECT OWNER,OBJECT_NAME,OBJECT_TYPE FROM DBA_OBJECTS WHERE OBJECT_NAME='DBA_DATA_FILES';

OWNER
------------------------------
OBJECT_NAME
--------------------------------------------------------------------------------
OBJECT_TYPE
-------------------
SYS
DBA_DATA_FILES
VIEW

PUBLIC
DBA_DATA_FILES
SYNONYM

OWNER
------------------------------
OBJECT_NAME
--------------------------------------------------------------------------------
OBJECT_TYPE
-------------------


SELECT * FROM DBA_SYNONYMS WHERE SYNONYM_NAME='DBA_DATA_FILES';

OWNER			       SYNONYM_NAME
------------------------------ ------------------------------
TABLE_OWNER		       TABLE_NAME
------------------------------ ------------------------------
DB_LINK
--------------------------------------------------------------------------------
PUBLIC			       DBA_DATA_FILES
SYS			       DBA_DATA_FILES

SQL> SELECT TEXT FROM DBA_VIEWS WHERE VIEW_NAME='DBA_DATA_FILES';

SQL> /
select v.name, f.file#, ts.name,
       ts.blocksize * f.blocks, f.blocks,
       decode(f.status$, 1, 'INVALID', 2, 'AVAILABLE', 'UNDEFINED'),
       f.relfile#, decode(f.inc, 0, 'NO', 'YES'),
       ts.blocksize * f.maxextend, f.maxextend, f.inc,
       ts.blocksize * (f.blocks - 1), f.blocks - 1,
       decode(fe.fetsn, 0, decode(bitand(fe.festa, 2), 0, 'SYSOFF', 'SYSTEM'),
	 decode(bitand(fe.festa, 18), 0, 'OFFLINE', 2, 'ONLINE', 'RECOVER'))
from sys.file$ f, sys.ts$ ts, sys.v$dbfile v, x$kccfe fe
where v.file# = f.file#
  and f.spare1 is NULL
  and f.ts# = ts.ts#
  and fe.fenum = f.file#
union all
select
       v.name,f.file#, ts.name,
       decode(hc.ktfbhccval, 0, ts.blocksize * hc.ktfbhcsz, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcsz, NULL),
       decode(f.status$, 1, 'INVALID', 2, 'AVAILABLE', 'UNDEFINED'),
       f.relfile#,
       decode(hc.ktfbhccval, 0, decode(hc.ktfbhcinc, 0, 'NO', 'YES'), NULL),
       decode(hc.ktfbhccval, 0, ts.blocksize * hc.ktfbhcmaxsz, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcmaxsz, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcinc, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcusz * ts.blocksize, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcusz, NULL),
       decode(fe.fetsn, 0, decode(bitand(fe.festa, 2), 0, 'SYSOFF', 'SYSTEM'),
	 decode(bitand(fe.festa, 18), 0, 'OFFLINE', 2, 'ONLINE', 'RECOVER'))
from sys.v$dbfile v, sys.file$ f, sys.x$ktfbhc hc, sys.ts$ ts, x$kccfe fe
where v.file# = f.file#
  and f.spare1 is NOT NULL
  and v.file# = hc.ktfbhcafno
  and hc.ktfbhctsn = ts.ts#
  and fe.fenum = f.file#

```







```
SQL> SET PAGES 0
SQL> /
select v.name, f.file#, ts.name,
       ts.blocksize * f.blocks, f.blocks,
       decode(f.status$, 1, 'INVALID', 2, 'AVAILABLE', 'UNDEFINED'),
       f.relfile#, decode(f.inc, 0, 'NO', 'YES'),
       ts.blocksize * f.maxextend, f.maxextend, f.inc,
       ts.blocksize * (f.blocks - 1), f.blocks - 1,
       decode(fe.fetsn, 0, decode(bitand(fe.festa, 2), 0, 'SYSOFF', 'SYSTEM'),
	 decode(bitand(fe.festa, 18), 0, 'OFFLINE', 2, 'ONLINE', 'RECOVER'))
from sys.file$ f, sys.ts$ ts, sys.v$dbfile v, x$kccfe fe
where v.file# = f.file#
  and f.spare1 is NULL
  and f.ts# = ts.ts#
  and fe.fenum = f.file#
union all
select
       v.name,f.file#, ts.name,
       decode(hc.ktfbhccval, 0, ts.blocksize * hc.ktfbhcsz, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcsz, NULL),
       decode(f.status$, 1, 'INVALID', 2, 'AVAILABLE', 'UNDEFINED'),
       f.relfile#,
       decode(hc.ktfbhccval, 0, decode(hc.ktfbhcinc, 0, 'NO', 'YES'), NULL),
       decode(hc.ktfbhccval, 0, ts.blocksize * hc.ktfbhcmaxsz, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcmaxsz, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcinc, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcusz * ts.blocksize, NULL),
       decode(hc.ktfbhccval, 0, hc.ktfbhcusz, NULL),
       decode(fe.fetsn, 0, decode(bitand(fe.festa, 2), 0, 'SYSOFF', 'SYSTEM'),
	 decode(bitand(fe.festa, 18), 0, 'OFFLINE', 2, 'ONLINE', 'RECOVER'))
from sys.v$dbfile v, sys.file$ f, sys.x$ktfbhc hc, sys.ts$ ts, x$kccfe fe
where v.file# = f.file#
  and f.spare1 is NOT NULL
  and v.file# = hc.ktfbhcafno
  and hc.ktfbhctsn = ts.ts#
  and fe.fenum = f.file#


SQL> 
```

