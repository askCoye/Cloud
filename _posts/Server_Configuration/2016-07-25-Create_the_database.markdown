---
layout: post
title:  "Create the Database"
date:   2016-07-25 07:48:57 +0000
comments: true
categories: Server_Configuration
---

## 通过命令创建/删除数据库 ##
oracle 官方文档 -> Oracle Database Administrator's Guide -> Creating and Configuring an Oracle Database ->[ Creating a Database with the CREATE DATABASE Statement](http://docs.oracle.com/cd/E11882_01/server.112/e25494/create.htm#ADMIN11074)

1. 登录Oracle用户

2. 指定SID（Instance Identifier）

    	export ORACLE_SID=OCM

3. 确保必须的环境变量已经设置，大多数平台上 ORACLE_SID 和 ORACLE_HOME 必须设置
		
		export ORACLE_SID=OCM
		export ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1

4. 选择 DBA 的认证方式
		
		1. 密码文件： password file
		2. 操作系统认证
		
5. 在 ORACLE_HOME/dbs 下创建静态参数文件（Initialization Parameter File）也就是initSID文件（[Sample Initialization Parameter File](http://docs.oracle.com/cd/E11882_01/server.112/e25494/create.htm#ADMIN11112)）
提前把init文件中的文件夹建好

		替换<ORACLE_BASE>
		db_name='ORCL'
		memory_target=1G
		processes = 150
		audit_file_dest='<ORACLE_BASE>/admin/orcl/adump'
		audit_trail ='db'
		db_block_size=8192
		db_domain=''
		db_recovery_file_dest='<ORACLE_BASE>/flash_recovery_area'
		db_recovery_file_dest_size=2G
		diagnostic_dest='/u01/app/oracle'
		dispatchers='(PROTOCOL=TCP) (SERVICE=ORCLXDB)'
		open_cursors=300 
		remote_login_passwordfile='EXCLUSIVE'
		undo_tablespace='UNDOTBS1'
		control_files = (ora_control1, ora_control2)
		compatible ='11.2.0'


6. 连上实例（Instance）

		1. 密码文件：
			$ sqlplus /nolog
			SQL> CONNECT SYS AS SYSDBA
			如果没有创建口令文件，随意数据密码均能连接
		2. 操作系统认证：
			$ sqlplus /nolog
			SQL> CONNECT / AS SYSDBA
7. 创建SPFILE
		
		SQL> CREATE SPFILE FROM PFILE;
		
8. 启动实例到NOMOUNT

		SQL> startup nomount;
		ORACLE instance started.
		
		Total System Global Area 1068937216 bytes
		Fixed Size		    2235208 bytes
		Variable Size		  616563896 bytes
		Database Buffers	  444596224 bytes
		Redo Buffers		    5541888 bytes

10. 执行创建数据库的语句（提前建好文件夹）
		
		替换密码sys_password
		替换UNDO表空间的名称
		CREATE DATABASE ORCL
		   USER SYS IDENTIFIED BY sys_password
		   USER SYSTEM IDENTIFIED BY system_password
		   LOGFILE GROUP 1 ('/u01/app/oracle/oradata/orcl/redo01a.log','/u02/logs/orcl/redo01b.log') SIZE 100M BLOCKSIZE 512,
		           GROUP 2 ('/u01/app/oracle/oradata/orcl/redo02a.log','/u02/logs/orcl/redo02b.log') SIZE 100M BLOCKSIZE 512,
		           GROUP 3 ('/u01/app/oracle/oradata/orcl/redo03a.log','/u02/logs/orcl/redo03b.log') SIZE 100M BLOCKSIZE 512
		   MAXLOGFILES 5
		   MAXLOGMEMBERS 5
		   MAXLOGHISTORY 1
		   MAXDATAFILES 100
		   CHARACTER SET AL32UTF8
		   NATIONAL CHARACTER SET AL16UTF16
		   EXTENT MANAGEMENT LOCAL
		   DATAFILE '/u01/app/oracle/oradata/orcl/system01.dbf' SIZE 325M REUSE
		   SYSAUX DATAFILE '/u01/app/oracle/oradata/orcl/sysaux01.dbf' SIZE 325M REUSE
		   DEFAULT TABLESPACE users
		      DATAFILE '/u01/app/oracle/oradata/orcl/users01.dbf'
		      SIZE 500M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED
		   DEFAULT TEMPORARY TABLESPACE tempts1
		      TEMPFILE '/u01/app/oracle/oradata/orcl/temp01.dbf'
		      SIZE 20M REUSE
		   UNDO TABLESPACE UNDOTBS1
		      DATAFILE '/u01/app/oracle/oradata/orcl/undotbs01.dbf'
		      SIZE 200M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;

11. 创建数据字典视图

		用 SYSDBA 权限执行
		@?/rdbms/admin/catalog.sql
		@?/rdbms/admin/catproc.sql
		@?/rdbms/admin/utlrp.sql
		用 SYSTEM 用户执行
		@?/sqlplus/admin/pupbld.sql

至此数据库安装完毕
查看数据库状态

		SQL> select open_mode from v$database;

		OPEN_MODE
		--------------------
		READ WRITE
		
		1 row selected.

删除数据库

	SQL> startup nomount;
	ORACLE instance started.
	
	Total System Global Area 1068937216 bytes
	Fixed Size                  2235208 bytes
	Variable Size             620758200 bytes
	Database Buffers          440401920 bytes
	Redo Buffers                5541888 bytes
	SQL> alter database mount exclusive;

	Database altered.
	
	SQL> alter system enable restricted session;
	
	System altered.
	
	SQL> drop database;
	
	Database dropped.
	
	Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.3.0 - 64bit Production
	With the Partitioning, OLAP, Data Mining and Real Application Testing options


## 通过DBCA创建数据库 ##
1. 登录oracle用户 

2. 输入 dbca

3. 
45. 进入到下一个屏幕 -> 点击 “Next”

5. 选择 “Create a Database” -> 点击 “Next”

6. 选择 “Custom Database” -> 点击 “Next”

7. 进入“Global Database Name” 输入 SID

    	Global Database Name = “OCM”
    	SID = “OCM”
    	点击 “Next”

8. 取消 “Configure Enterpris Manager” -> 点击 “Next”

9. 选择 “Use the Same Administrative Password for All Account”

    	Password = “xxxxxx”
    	Confirm Password = “xxxxxx”
    	点击 “Next”

10. 对于存储类型:

    	Storage Type = “File System”
    	选择 “Use Common Location for All Database Files”
    	Database Files Location = “{ORACLE_BASE}/oradata”
    	点击 “Next”	

11. 我们选择非默认位置 Flash/Fast Recovery Area -> 选择 “Enable Archiving” -> 点击 “Next”

12. 我们保留默认组件 -> 点击 “Next”

13. 选择Unicode字符集，其他选项默认

    	点击选项卡 “Character Sets”
    	点击 “Use Unicode (AL32UTF8)
    	点击 “Next”

14. 在窗口 “Database Storage” -> 点击 “Next”

15. 勾选 “Create Database” 和 “Generate Database Creation Scripts” -> 点击 “Next”

16. 确认摘要 -> 点击 “OK”

17. 我们将得到创建数据库的脚本 -> 点击 “OK”

18. 一旦安装完成 -> 点击 “Exit”

19. 我们不仅处于在ARCHIVELOG模式下数据库中，我们趁机也激活闪回。 闪回模式允许我们回到数据库的更早的时刻。

    	sqlplus / as sysdba
    	alter system set db_recovery_file_dest='/u01/app/oracle/flash_recovery_area' scope=spfile
    	alter system set db_recovery_file_desc_size=2G
    	SHUTDOWN IMMEDIATE;
    	STARTUP MOUNT;
    	ALTER DATABASE ARCHIVELOG;
    	ALTER DATABASE FLASHBACK ON;
    	ALTER DATABASE OPEN;

20. 检查设置

    	sqlplus / as sysdba
    	SELECT NAME, LOG_MODE, FLASHBACK_ON FROM V$DATABASE;