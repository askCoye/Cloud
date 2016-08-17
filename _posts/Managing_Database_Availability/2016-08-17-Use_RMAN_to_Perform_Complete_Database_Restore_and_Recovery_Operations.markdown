---
layout: post
title:  "4 Use RMAN to Perform Complete Database Restore and Recovery Operations"
date:   2016-08-17 15:48:57 +0000
comments: true
categories: Managing_Database_Availability
---

# 4 Use RMAN to Perform Complete Database Restore and Recovery Operations #

1. 官方文档 -> Masters Book List -> Backup and Recovery User’s Guide -> [17 Performing Complete Database Recovery](http://docs.oracle.com/cd/E11882_01/backup.112/e10642/rcmcomre.htm#BRADV8005)

	官方文档 -> Masters Book List -> Backup and Recovery User’s Guide -> [30 Performing User-Managed Recovery: Advanced Scenarios](http://docs.oracle.com/cd/E11882_01/backup.112/e10642/osadvsce.htm#BRADV018)
2. 保证数据库的有效备份，删除属于SYSTEM表空间数据文件1，进行恢复。

		--修改system01.dbf 文件
	
		[root@oracle ocm]# mv system01.dbf system01.dbf.org
	
		--关闭并且重新启动数据库
		SQL> startup force   
		ORACLE instance started.
	
		Total System Global Area  430075904 bytes
		Fixed Size		    2229064 bytes
		Variable Size		  322964664 bytes
		Database Buffers	   96468992 bytes
		Redo Buffers		    8413184 bytes
		Database mounted.
		ORA-01157: cannot identify/lock data file 1 - see DBWR trace file
		ORA-01110: data file 1: '/u01/app/oracle/oradata/ocm/system01.dbf'
		
		--无法实验catalog连接rman
		[oracle@oracle ~]$ rman target sys/oracle@OCM catalog rman/rman@OCM
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Wed Jul 13 08:55:27 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2299627592, not open)
		RMAN-00571: ===========================================================
		RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
		RMAN-00571: ===========================================================
		RMAN-00554: initialization of internal recovery manager package failed
		RMAN-04004: error from recovery catalog database: ORA-01033: ORACLE initialization or shutdown in progress
		
		--直连rman
		[oracle@oracle ~]$ rman target /
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Wed Jul 13 08:55:41 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2299627592, not open)
	
		--恢复数据文件，并打开数据库
		RMAN> RESTORE DATAFILE 1;
	
		Starting restore at 13-JUL-2016 08:55:48
		using target database control file instead of recovery catalog
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=136 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=11 device type=DISK
	
		channel ORA_DISK_1: starting datafile backup set restore
		channel ORA_DISK_1: specifying datafile(s) to restore from backup set
		channel ORA_DISK_1: restoring datafile 00001 to /u01/app/oracle/oradata/ocm/system01.dbf
		channel ORA_DISK_1: reading from backup piece /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T085057_crc43ny8_.bkp
		channel ORA_DISK_1: piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T085057_crc43ny8_.bkp tag=TAG20160713T085057
		channel ORA_DISK_1: restored backup piece 1
		channel ORA_DISK_1: restore complete, elapsed time: 00:00:16
		Finished restore at 13-JUL-2016 08:56:07
	
		RMAN> RECOVER DATAFILE 1;
	
		Starting recover at 13-JUL-2016 08:56:12
		using channel ORA_DISK_1
		using channel ORA_DISK_2
	
		starting media recovery
		media recovery complete, elapsed time: 00:00:01
	
		Finished recover at 13-JUL-2016 08:56:14
	
		RMAN> sql 'ALTER DATABASE OPEN';
	
		sql statement: ALTER DATABASE OPEN

3. 使用Data Recovery Advisor (DRA)的新功能。修复丢失的数据文件。

		--模拟移除Users表空间的数据文件 users01.dbf 文件
	
		[root@oracle ocm]# mv users01.dbf users01.dbf.org
	
		--在Users表空间上创建表
		SQL> create table x (a char(100)) tablespace USERS;
		create table x (a char(100)) tablespace USERS
		*
		ERROR at line 1:
		ORA-01116: error in opening database file 4
		ORA-01110: data file 4: '/u01/app/oracle/oradata/ocm/users01.dbf'
		ORA-27041: unable to open file
		Linux-x86_64 Error: 2: No such file or directory
		Additional information: 3
	
		--通过RMAN连接数据库和catalog
		[oracle@oracle ~]$ rman target sys/oracle@OCM catalog rman/rman@OCM
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Wed Jul 13 09:06:05 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2299627592)
		connected to recovery catalog database
		
		--列出DB中的问题
		RMAN> list failure;
	
		List of Database Failures
		=========================
	
		Failure ID Priority Status    Time Detected        Summary
		---------- -------- --------- -------------------- -------
		1282       HIGH     OPEN      13-JUL-2016 09:03:18 One or more non-system datafiles are missing
	
		--DRA分析问题产生的原因，以及给出修复脚本
		RMAN> advise failure;
	
		List of Database Failures
		=========================
	
		Failure ID Priority Status    Time Detected        Summary
		---------- -------- --------- -------------------- -------
		1282       HIGH     OPEN      13-JUL-2016 09:03:18 One or more non-system datafiles are missing
	
		analyzing automatic repair options; this may take some time
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=10 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=146 device type=DISK
		analyzing automatic repair options complete
	
		Mandatory Manual Actions
		========================
		no manual actions available
	
		Optional Manual Actions
		=======================
		1. If file /u01/app/oracle/oradata/ocm/users01.dbf was unintentionally renamed or moved, restore it
	
		Automated Repair Options
		========================
		Option Repair Description
		------ ------------------
		1      Restore and recover datafile 4  
	 	 Strategy: The repair includes complete media recovery with no data loss
		  Repair script: /u01/app/oracle/diag/rdbms/ocm/ocm/hm/reco_1281316010.hm
		
		--预览修复过程以及脚本
		RMAN> repair failure preview;
	
		Strategy: The repair includes complete media recovery with no data loss
		Repair script: /u01/app/oracle/diag/rdbms/ocm/ocm/hm/reco_1281316010.hm
	
		contents of repair script:
		   # restore and recover datafile
		   sql 'alter database datafile 4 offline';
	 	  restore datafile 4;
	 	  recover datafile 4;
	 	  sql 'alter database datafile 4 online';
	
		--执行修复脚本
		RMAN> repair failure;
	
		Strategy: The repair includes complete media recovery with no data loss
		Repair script: /u01/app/oracle/diag/rdbms/ocm/ocm/hm/reco_1281316010.hm
	
		contents of repair script:
		   # restore and recover datafile
		   sql 'alter database datafile 4 offline';
		   restore datafile 4;
		   recover datafile 4;
	 	  sql 'alter database datafile 4 online';
	
		Do you really want to execute the above repair (enter YES or NO)? yes
		executing repair script
	
		sql statement: alter database datafile 4 offline
	
		Starting restore at 13-JUL-2016 09:08:02
		using channel ORA_DISK_1
		using channel ORA_DISK_2
	
		channel ORA_DISK_1: starting datafile backup set restore
		channel ORA_DISK_1: specifying datafile(s) to restore from backup set
		channel ORA_DISK_1: restoring datafile 00004 to /u01/app/oracle/oradata/ocm/users01.dbf
		channel ORA_DISK_1: reading from backup piece /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T085057_crc43o0x_.bkp
		channel ORA_DISK_1: piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T085057_crc43o0x_.bkp tag=TAG20160713T085057
		channel ORA_DISK_1: restored backup piece 1
		channel ORA_DISK_1: restore complete, elapsed time: 00:00:01
		Finished restore at 13-JUL-2016 09:08:04
	
		Starting recover at 13-JUL-2016 09:08:04
		using channel ORA_DISK_1
		using channel ORA_DISK_2
	
		starting media recovery
		media recovery complete, elapsed time: 00:00:00
	
		Finished recover at 13-JUL-2016 09:08:04
	
		sql statement: alter database datafile 4 online
		repair failure complete
	
5. 数据文件丢失以后，在另一个位置恢复出来。
	
		--模拟移除Users表空间的数据文件 users01.dbf 文件
		[root@oracle ocm]# mv users01.dbf users01.dbf.org
	
		--在Users表空间上创建表
		SQL> create table x (a char(100)) tablespace USERS;
		create table x (a char(100)) tablespace USERS
		*
		ERROR at line 1:
		ORA-01116: error in opening database file 4
		ORA-01110: data file 4: '/u01/app/oracle/oradata/ocm/users01.dbf'
		ORA-27041: unable to open file
		Linux-x86_64 Error: 2: No such file or directory
		Additional information: 3
	
		--修复
	
		[oracle@oracle ~]$ rman target /
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Wed Jul 13 09:28:23 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2299627592, not open)
	
		RMAN> 	RUN {
	 	 	sql 'ALTER DATABASE DATAFILE 4 OFFLINE';
	  		SET NEWNAME FOR DATAFILE 4 TO '/u01/users01.dbf';
	  		RESTORE DATAFILE 4;
	  		SWITCH DATAFILE 4;
	  		RECOVER DATAFILE 4;
	  		sql 'ALTER DATABASE DATAFILE 4 ONLINE';
			}2> 3> 4> 5> 6> 7> 8> 
	
		using target database control file instead of recovery catalog
		sql statement: ALTER DATABASE DATAFILE 4 OFFLINE
	
		executing command: SET NEWNAME
	
		Starting restore at 13-JUL-2016 09:28:27
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=135 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=11 device type=DISK
	
		channel ORA_DISK_1: starting datafile backup set restore
		channel ORA_DISK_1: specifying datafile(s) to restore from backup set
		channel ORA_DISK_1: restoring datafile 00004 to /u01/users01.dbf
		channel ORA_DISK_1: reading from backup piece /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T085057_crc43o0x_.bkp
		channel ORA_DISK_1: piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T085057_crc43o0x_.bkp tag=TAG20160713T085057
		channel ORA_DISK_1: restored backup piece 1
		channel ORA_DISK_1: restore complete, elapsed time: 00:00:01
		Finished restore at 13-JUL-2016 09:28:30
	
		datafile 4 switched to datafile copy
		input datafile copy RECID=15 STAMP=917083710 file name=/u01/users01.dbf
	
		Starting recover at 13-JUL-2016 09:28:30
		using channel ORA_DISK_1
		using channel ORA_DISK_2
	
		starting media recovery
		media recovery complete, elapsed time: 00:00:00
	
		Finished recover at 13-JUL-2016 09:28:30
	
		sql statement: ALTER DATABASE DATAFILE 4 ONLINE

6. 如果数据文件丢失了，也没有数据文件的备份，依然可以恢复数据文件。前提是必须拥有全部的归档日志，而且数据文件的名字必须包含在控制文件里面。

		--创建一个表空间TEST
		--在表空间TEST中创建表test_table
	
		SQL> create tablespace test datafile '/u01/app/oracle/oradata/ocm/test01.dbf' size 100m;
	
		Tablespace created.
	
		SQL> create table test_table tablespace test as select * from dba_tables;
	
		Table created.
	
		--删除数据文件
		[oracle@oracle ocm]$ rm -rf /u01/app/oracle/oradata/ocm/test01.dbf 
		--Offine数据文件
		SQL> ALTER DATABASE DATAFILE 6 OFFLINE;
	
		Database altered.
		--创建一个和删除的数据文件一样的数据文件
		SQL> ALTER DATABASE CREATE DATAFILE '/u01/app/oracle/oradata/ocm/test01.dbf' AS '/u01/app/oracle/oradata/ocm/test01.dbf';
	
		Database altered.
	
		--RECOVER 数据文件
	
		SQL> RECOVER DATAFILE 6;
		Media recovery complete.
	
		--ONLINE数据文件
		SQL> ALTER DATABASE DATAFILE 6 ONLINE;
	
		Database altered.
		--打开数据库
		SQL> alter database open;
	
		Database altered.
	
		SQL> SELECT COUNT(*) FROM TEST_TABLE;
	
	 	 COUNT(*)
		----------
	      		2816
		--删除测试表空间
		SQL> DROP TABLESPACE TEST INCLUDING CONTENTS AND DATAFILES;
	
		Tablespace dropped.

