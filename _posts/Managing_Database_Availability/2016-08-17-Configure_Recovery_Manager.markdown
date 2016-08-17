---
layout: post
title:  "2 Configure Recovery Manager"
date:   2016-08-17 12:48:57 +0000
comments: true
categories: Managing_Database_Availability
---

# 2 Configure Recovery Manager  #

1. 官方文档 -> Masters Book List -> Backup and Recovery User’s Guide -> [5 Configuring the RMAN Environment](http://docs.oracle.com/cd/E11882_01/backup.112/e10642/rcmconfb.htm#BRADV8002)

2. 连接RMAN，并查看默认参数

		[oracle@ocm ~]$ rman target sys/oracle@OCM catalog rman/rman@OCM
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Sun Jul 10 14:57:07 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2295194131)
		connected to recovery catalog database
	
		RMAN> show all;
	
		RMAN configuration parameters for database with db_unique_name OCM are:
		CONFIGURE RETENTION POLICY TO REDUNDANCY 1; # default
		CONFIGURE BACKUP OPTIMIZATION OFF; # default
		CONFIGURE DEFAULT DEVICE TYPE TO DISK; # default
		CONFIGURE CONTROLFILE AUTOBACKUP OFF; # default
		CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
		CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; # default
		CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
		CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
		CONFIGURE MAXSETSIZE TO UNLIMITED; # default
		CONFIGURE ENCRYPTION FOR DATABASE OFF; # default
		CONFIGURE ENCRYPTION ALGORITHM 'AES128'; # default
		CONFIGURE COMPRESSION ALGORITHM 'BASIC' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE ; # default
		CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default
		CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/u01/app/oracle/product/11.2.0/dbhome_1/dbs/snapcf_ocm.f'; # default
	
		RMAN> 

3. 设置自动备份模式，执行备份的时候、数据库结构发生变化的时候会自动备份控制文件。

		RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;  
	
		new RMAN configuration parameters:
		CONFIGURE CONTROLFILE AUTOBACKUP ON;
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete

		--当我们设置了快速恢复区，默认情况下，备份的 CONTROLFILE 和 SPFILE 就存放在里面。<fast_recovery_area>/ OCM/autobackup/*
		--来更改 AUTOBACKUP 的默认路径

		RMAN> CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/u01/%F';
	
		new RMAN configuration parameters:
		CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/u01/%F';
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete

		--恢复默认路径到FRA（官档3.4.1.2 Restoring Default RMAN Configuration Settings: CONFIGURE... CLEAR）

		RMAN> CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK CLEAR;
	
	
		old RMAN configuration parameters:
		CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '/u01/%F';
		RMAN configuration parameters are successfully reset to default value
		starting full resync of recovery catalog
		full resync complete
	
		RMAN> show all;
	
		RMAN configuration parameters for database with db_unique_name OCM are:
		CONFIGURE RETENTION POLICY TO REDUNDANCY 1; # default
		CONFIGURE BACKUP OPTIMIZATION OFF; # default
		CONFIGURE DEFAULT DEVICE TYPE TO DISK; # default
		CONFIGURE CONTROLFILE AUTOBACKUP ON;
		CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
		CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; # default
		CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
		CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
		CONFIGURE MAXSETSIZE TO UNLIMITED; # default
		CONFIGURE ENCRYPTION FOR DATABASE OFF; # default
		CONFIGURE ENCRYPTION ALGORITHM 'AES128'; # default
		CONFIGURE COMPRESSION ALGORITHM 'BASIC' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE ; # default
		CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default
		CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/u01/app/oracle/product/11.2.0/dbhome_1/dbs/snapcf_ocm.f'; # default
4. RMAN保留策略。允许控制备份的保留时间。根据业务需求。可以指定一个保留天数，或者可以指定我们的备份（冗余）要多少份保存（RECOVERY WINDOW）。默认情况下，我们有一个冗余副本。

		--提高备份的拷贝数
		RMAN> CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
	
		new RMAN configuration parameters:
		CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete
	
		--禁用保留策略（RMAN不考虑任何备份为过时）
		RMAN> CONFIGURE RETENTION POLICY TO NONE;
	
		old RMAN configuration parameters:
		CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
		new RMAN configuration parameters:
		CONFIGURE RETENTION POLICY TO NONE;
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete
	
		--恢复默认（官档3.4.1.2 Restoring Default RMAN Configuration Settings: CONFIGURE... CLEAR）
		RMAN> 	CONFIGURE RETENTION POLICY CLEAR;
	
		old RMAN configuration parameters:
		CONFIGURE RETENTION POLICY TO NONE;
		RMAN configuration parameters are successfully reset to default value
		starting full resync of recovery catalog
		full resync complete
	
		RMAN> show all;
	
		RMAN configuration parameters for database with db_unique_name OCM are:
		CONFIGURE RETENTION POLICY TO REDUNDANCY 1; # default
		CONFIGURE BACKUP OPTIMIZATION OFF; # default
		CONFIGURE DEFAULT DEVICE TYPE TO DISK; # default
		CONFIGURE CONTROLFILE AUTOBACKUP ON;
		CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
		CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; # default
		CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
		CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
		CONFIGURE MAXSETSIZE TO UNLIMITED; # default
		CONFIGURE ENCRYPTION FOR DATABASE OFF; # default
		CONFIGURE ENCRYPTION ALGORITHM 'AES128'; # default
		CONFIGURE COMPRESSION ALGORITHM 'BASIC' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE ; # default
		CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default
		CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/u01/app/oracle/product/11.2.0/dbhome_1/dbs/snapcf_ocm.f'; # default
		
		--设置备份策略保留7天
		--重要的是，这个保留参数小于 CONTROL_FILE_RECORD_KEEP_TIME
		RMAN> CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
	
		new RMAN configuration parameters:
		CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete

5. 使用OPTIMIZATION时，在一定的条件下发生的RMAN备份，可跳过在读模式下的表空间。（Using Backup Optimization to Skip Files）

		RMAN> CONFIGURE BACKUP OPTIMIZATION ON;
	
		new RMAN configuration parameters:
		CONFIGURE BACKUP OPTIMIZATION ON;
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete

6. 在我们的脚本中，我们可以为备份指定打开一个或多个信道。我们可以指定一个通道的默认值。
	
		--我们还可以通过parallelism参数来指定同时"自动"创建2个通道：
		RMAN> CONFIGURE DEVICE TYPE DISK PARALLELISM 2 BACKUP TYPE TO BACKUPSET;
	
		new RMAN configuration parameters:
		CONFIGURE DEVICE TYPE DISK PARALLELISM 2 BACKUP TYPE TO BACKUPSET;
		new RMAN configuration parameters are successfully stored
		starting full resync of recovery catalog
		full resync complete
	
		--备份数据文件，测试是否自动打开两个通道
		RMAN> BACKUP DATAFILE 4,5;
	
		Starting backup at 10-JUL-2016 15:56:18
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=24 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=23 device type=DISK
		channel ORA_DISK_1: starting full datafile backup set
		channel ORA_DISK_1: specifying datafile(s) in backup set
		input datafile file number=00005 name=/u01/app/oracle/oradata/ocm/rcat.dbf
		channel ORA_DISK_1: starting piece 1 at 10-JUL-2016 15:56:19
		channel ORA_DISK_2: starting full datafile backup set
		channel ORA_DISK_2: specifying datafile(s) in backup set
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		channel ORA_DISK_2: starting piece 1 at 10-JUL-2016 15:56:19
		channel ORA_DISK_1: finished piece 1 at 10-JUL-2016 15:56:20
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_10/o1_mf_nnndf_TAG20160710T155619_cr3zx3t5_.bkp tag=TAG20160710T155619 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		channel ORA_DISK_2: finished piece 1 at 10-JUL-2016 15:56:20
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_10/o1_mf_nnndf_TAG20160710T155619_cr3zx3t9_.bkp tag=TAG20160710T155619 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:01
		Finished backup at 10-JUL-2016 15:56:20
	
		Starting Control File and SPFILE Autobackup at 10-JUL-2016 15:56:20
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_10/o1_mf_s_916847781_cr3zx5nf_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 10-JUL-2016 15:56:22
	
		--删除备份
	
		RMAN> DELETE BACKUP OF DATAFILE 4,5;
	
		using channel ORA_DISK_1
		using channel ORA_DISK_2
	
		List of Backup Pieces
		BP Key  BS Key  Pc# Cp# Status      Device Type Piece Name
		------- ------- --- --- ----------- ----------- ----------
		202     200     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_10/o1_mf_nnndf_TAG20160710T155619_cr3zx3t9_.bkp
		203     201     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_10/o1_mf_nnndf_TAG20160710T155619_cr3zx3t5_.bkp
	
		Do you really want to delete the above objects (enter YES or NO)? yes
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_10/o1_mf_nnndf_TAG20160710T155619_cr3zx3t9_.bkp RECID=1 STAMP=916847779
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_10/o1_mf_nnndf_TAG20160710T155619_cr3zx3t5_.bkp RECID=2 STAMP=916847779
		Deleted 2 objects