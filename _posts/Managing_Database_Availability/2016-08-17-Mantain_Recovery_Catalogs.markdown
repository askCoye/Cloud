---
layout: post
title:  "Mantain Recovery Catalogs"
date:   2016-08-17 10:48:57 +0000
comments: true
categories: Managing_Database_Availability
---

# 1 Mantain Recovery Catalogs  #

1. 官方文档 -> Masters Book List -> Backup and Recovery User's Guide -> 13 [Managing a Recovery Catalog](http://docs.oracle.com/cd/E11882_01/backup.112/e10642/rcmcatdb.htm#i1011365)

2. 作为一个DBA的最重要的是不丢失数据(呵呵，曾经有同事把/u01下面的数据库连同数据文件一起 rm -rf了，没做备份。。。连条毛都恢复不了)。

3. 用OCM数据库保存 RMAN 恢复目录（recovery catalog）.

4. 先检查OCM库

		-- 是否归档模式
		SQL> archive log list    
		Database log mode	       Archive Mode
		Automatic archival	       Enabled
		Archive destination	       USE_DB_RECOVERY_FILE_DEST
		Oldest online log sequence     39
		Next log sequence to archive   41
		Current log sequence	       41

		-- FLASHBACK 模式是否开启
		SQL> select flashback_on from v$database;
	
		FLASHBACK_ON
		------------------
		NO
		-- 如果没开启，需要设置和启用FRA（fast_recovery_area）
		SQL> show parameter recovery_file
	
		NAME				     TYPE	 VALUE
		------------------------------------ ----------- ------------------------------
		db_recovery_file_dest		     string
		db_recovery_file_dest_size	     big integer 0
		
		SQL> alter system set db_recovery_file_dest_size=4G scope=both;
	
		System altered.
	
		SQL> alter system set db_recovery_file_dest='/u01/app/oracle/flash_recovery_area' scope=both;
	
		System altered.
	
		SQL> show parameter recovery_file
	
		NAME						TYPE			VALUE
		------------------------------------ 	----------- 		------------------------------
		db_recovery_file_dest		    	string			/u01/app/oracle/flash_recovery_area
		db_recovery_file_dest_size		big integer 	4122M
		SQL> alter database flashback on;
	
		Database altered.
	
		SQL> select flashback_on from v$database;
	
		FLASHBACK_ON
		------------------
		YES
		--打开增量备份块更改跟踪功能
		--默认是由参数DB_CREATE_FILE_DEST指定的创建位置
		alter database enable BLOCK CHANGE TRACKING using file '/u01/app/oracle/oradata/ocm/block_change_tracking.f' reuse;
		--查看是否成功创建
		SQL> select * from v$block_change_tracking;
	
		STATUS		FILENAME													BYTES
		----------		--------------------------------------------------------------------------------	----------
		ENABLED		/u01/app/oracle/oradata/ocm/block_change_tracking.f	  			11599872


5. 创建恢复目录（Recovery Catalog）.

		--创建恢复目录所需的表空间
		CREATE  TABLESPACE RCAT DATAFILE '/u01/app/oracle/oradata/ocm/rcat.dbf'
		SIZE 100M AUTOEXTEND ON NEXT 1M MAXSIZE 1G EXTENT MANAGEMENT LOCAL UNIFORM SIZE 1M;

		--创建恢复目录的所有者，以及赋予相应权限
		CREATE  USER  rman Identified BY  RMAN DEFAULT  TABLESPACE RCAT TEMPORARY  TABLESPACE TEMP ;
		ALTER  USER  rman QUOTA UNLIMITED ON  RCAT;
		Grant  RECOVERY_CATALOG_OWNER TO  rman;

		--连接RMAN 的catalog 并创建catalog
		[oracle@ocm ocm]$ rman catalog rman/rman@OCM
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Sun Jul 10 10:25:08 2016
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
		connected to recovery catalog database
	
		RMAN> create catalog;
	
		recovery catalog created

		--重新连接RMAN，然后注册OCM数据库
		[oracle@oracle ~]$ rman target sys/oracle@OCM catalog rman/rman@OCM
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Tue Jul 12 15:53:01 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2299627592)
		connected to recovery catalog database
	
		RMAN> register database ;
	
		database registered in recovery catalog
		starting full resync of recovery catalog
		full resync complete


6. 在恢复目录中记录OEM
		--编辑TNSNAMES.ORA
		vi $ORACLE_HOME/network/admin/tnsnames.ora
	
		--增加
		OCM =
	  	(DESCRIPTION =
	   	 (ADDRESS_LIST =
	   	   (ADDRESS = (PROTOCOL = TCP)(HOST = ocm.oracle.com)(PORT = 1521))
	   	 )
	   	 (CONNECT_DATA =
	   	   (SERVICE_NAME = ocm.oracle.com)
	   	 )
	  	)
	
		--在恢复目录中记录OEM
		[oracle@oem admin]$ rman target sys@oem catalog rman/rman@ocm
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Sun Jul 10 10:55:18 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		target database Password: 
		connected to target database: OEM (DBID=2759854072)
		connected to recovery catalog database
	
		RMAN> register database;
	
		database registered in recovery catalog
		starting full resync of recovery catalog
		full resync complete
	
7. 注册数据库已经作出一个隐含的同步（重新同步），也可以手动执行。
		RMAN> resync catalog;
	
		starting full resync of recovery catalog
		full resync complete