---
layout: post
title:  "Determine and Set Sizing Parameters for Database Structures"
date:   2016-08-05 10:48:57 +0000
comments: true
categories: Server_Configuration
---

# Determine and Set Sizing Parameters for Database Structures  #
## 确定和设置数据库结构尺寸参数 ##

1. oracle 官方文档 ->  Masters Book List -> Administrator’s Guide -> 2 Creating and Configuring an Oracle Database -> [Specifying Initialization Parameters](http://docs.oracle.com/cd/E11882_01/server.112/e25494/create.htm#CIAFAFFG)

2. 查看和定位数据库的各种参数
		
		登录Oracle
		[oracle@oracle ~]$ sqlplus / as sysdba

		SQL*Plus: Release 11.2.0.3.0 Production on Fri Aug 5 03:11:11 2016
		
		Copyright (c) 1982, 2011, Oracle.  All rights reserved.
		
		Connected to:
		Oracle Database 11g Enterprise Edition Release 11.2.0.3.0 - 64bit Production
		With the Partitioning, OLAP, Data Mining and Real Application Testing options
		
		查看当前会话的参数
		SQL> show parameter
		
		查看指定的参数
		SQL> show parameter db_recovery
		NAME				     				TYPE	 	VALUE
		------------------------------------ ----------- ------------------------------
		db_recovery_file_dest		     		string	 	/u01/app/oracle/flash_recovery_area
		db_recovery_file_dest_size	     		big integer 12G
		
		这个命令等同于以下查询
		SQL> COL NAME FORMAT A35
		SQL> COL VALUE FORMAT A40
		SQL> SELECT NAME, VALUE FROM V$PARAMETER WHERE LOWER(NAME) LIKE '%db_recovery%';
		NAME				    			VALUE
		----------------------------------- ----------------------------------------
		db_recovery_file_dest		    	/u01/app/oracle/flash_recovery_area
		db_recovery_file_dest_size	    	12884901888
		
		查看启动实例使用的是SPFILE还是PFILE
		可以查看SPFILE的位置，如果是通过PFILE启动的，则为空
		SQL> show parameter spfile
		NAME				     				TYPE	 	VALUE
		------------------------------------ ----------- ------------------------------
		spfile				     				string	 /u01/app/oracle/product/11.2.0
								 						 /dbhome_1/dbs/spfileORCL.ora
		如果是用SPFILE启动的，可以用以下方式查询
		SQL> show spparameters db_recovery
		SID	 		NAME			       		TYPE	   VALUE
		-------- ----------------------------- ----------- ----------------------------
			*	 db_recovery_file_dest	       string	   /u01/app/oracle/flash_recovery_area
			*	 db_recovery_file_dest_size    big integer 12G

		此查询使用的是V$SPPARAMETER，因此可以将此查询改写为以下SQL
		SQL> COL SID FORMAT A5
		SQL> COL NAME FORMAT A30
		SQL> SELECT SID,NAME, VALUE FROM V$SPPARAMETER WHERE LOWER(NAME) LIKE '%db_recovery%';
		
		SID   NAME			     				VALUE
		----- ------------------------------ ----------------------------------------
		*     db_recovery_file_dest	     		/u01/app/oracle/flash_recovery_area
		*     db_recovery_file_dest_size     	12884901888



2. Global Database Name

	全局数据库名称（Global Database Name）由用户指定的本地数据库名称和数据库的网络结构中的位置的。该DB_NAME初始化参数决定数据库名称的本地名称部分，而DB_DOMAIN参数，它是可选的，表示网络结构中的域（逻辑位置）。对于这两个参数的设置的组合必须形成一个数据库名称是在网络内是唯一的。  
		
	DB_NAME Initialization Parameter

	DB_NAME must be set to a text string of no more than eight characters. During database creation, the name provided for DB_NAME is recorded in the data files, redo log files, and control file of the database. If during database instance startup the value of the DB_NAME parameter (in the parameter file) and the database name in the control file are different, the database does not start.
	
	DB_DOMAIN Initialization Parameter

	DB_DOMAIN is a text string that specifies the network domain where the database is created. If the database you are about to create will ever be part of a distributed database system, then give special attention to this initialization parameter before database creation. This parameter is optional.
	
		查看db_name和db_domain的值
		SQL> show parameter db_name
		NAME				     				TYPE	 VALUE
		------------------------------------ ----------- ------------------------------
		db_name 			     				string	 ORCL

		SQL> show parameter db_domain		
		NAME				     				TYPE	 VALUE
		------------------------------------ ----------- ------------------------------
		db_domain			     				string	 oracle.com

		GLOBAL_DB_NAME等于db_name.db_domain
		SQL> SELECT PROPERTY_NAME, PROPERTY_VALUE FROM DATABASE_PROPERTIES WHERE PROPERTY_NAME = 'GLOBAL_DB_NAME';
		PROPERTY_NAME	     PROPERTY_VALUE
		-------------------- ------------------------------
		GLOBAL_DB_NAME	     ORCL.ORACLE.COM


3. 指定一个Fast Recovery Area

	Fast Recovery Area是oracle DB存储并管理备份和恢复相关文件的位置。它与数据库中当前数据库文件（数据文件，控制文件和redo log）的位置不同。
	
		指定Fast Recovery Area 主要用到以下两个参数：
		DB_RECOVERY_FILE_DEST: Location of the Fast Recovery Area. This can be a directory, file system, or 
		Automatic Storage Management (Oracle ASM) disk group. It cannot be a raw file system.
		在RAC环境中，这个位置必须在集群文件系统中，OracleASM磁盘组，或者通过NFS的共享目录
		DB_RECOVERY_FILE_DEST_SIZE: Specifies the maximum total bytes to be used by the Fast Recovery Area. This initialization parameter must be specified before DB_RECOVERY_FILE_DEST is enabled. 

		
		
