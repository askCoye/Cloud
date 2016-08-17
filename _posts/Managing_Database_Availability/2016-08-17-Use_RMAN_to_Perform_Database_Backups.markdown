---
layout: post
title:  "3 Use RMAN to Perform Database Backups"
date:   2016-08-17 15:48:57 +0000
comments: true
categories: Managing_Database_Availability
---


1. 官方文档 -> Masters Book List -> Backup and Recovery User’s Guide -> [9 Backing Up the Database](http://docs.oracle.com/cd/E11882_01/backup.112/e10642/rcmbckba.htm#BRADV8003)
2. 完整的备份与归档日志

		--用恢复目录（recovery catalog）连接OCM数据库
	
		[oracle@oracle ~]$ rman target sys/oracle@OCM catalog rman/rman@OCM
	
		Recovery Manager: Release 11.2.0.3.0 - Production on Wed Jul 13 06:11:50 2016
	
		Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
	
		connected to target database: OCM (DBID=2299627592)
		connected to recovery catalog database
	
		--数据库的全量备份（包括归档日志）
		RMAN> backup database plus archivelog;
	
	
		Starting backup at 13-JUL-2016 06:15:51
		current log archived
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=149 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=21 device type=DISK
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=5 RECID=1 STAMP=917055252
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:15:56
		channel ORA_DISK_2: starting archived log backup set
		channel ORA_DISK_2: specifying archived log(s) in backup set
		input archived log thread=1 sequence=6 RECID=2 STAMP=917072152
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 06:15:56
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:16:00
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061555_crbv0wtt_.bkp tag=TAG20160713T061555 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:04
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 06:16:00
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061555_crbv0x1n_.bkp tag	=TAG20160713T061555 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:04
		Finished backup at 13-JUL-2016 06:16:00
	
		Starting backup at 13-JUL-2016 06:16:00
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting full datafile backup set
		channel ORA_DISK_1: specifying datafile(s) in backup set
		input datafile file number=00002 name=/u01/app/oracle/oradata/ocm/sysaux01.dbf
		input datafile file number=00005 name=/u01/app/oracle/oradata/ocm/rcat.dbf
		input datafile file number=00003 name=/u01/app/oracle/oradata/ocm/undotbs01.dbf
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:16:03
		channel ORA_DISK_2: starting full datafile backup set
		channel ORA_DISK_2: specifying datafile(s) in backup set
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 06:16:03
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 06:17:28
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T061600_crbv139h_.bkp tag=TAG20160713T061600 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:01:25
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:17:38
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T061600_crbv13c3_.bkp tag=TAG20160713T061600 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:01:35
		Finished backup at 13-JUL-2016 06:17:38
	
		Starting backup at 13-JUL-2016 06:17:38
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=7 RECID=3 STAMP=917072258
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:17:43
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:17:44
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061741_crbv47cy_.bkp tag=TAG20160713T061741 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 06:17:44
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:17:45
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917072267_crbv4cmo_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:17:48

		--默认情况下，备份是在FRA下面
		SQL> select * from v$recovery_area_usage;
	
		FILE_TYPE	     		PERCENT_SPACE_USED 	PERCENT_SPACE_RECLAIMABLE	NUMBER_OF_FILES
		-------------------- ------------------ -------------------------	---------------
		CONTROL FILE			  0 			0					  		0
		REDO LOG			      0 			0							0
		ARCHIVED LOG			  1.58 		    1.58						3
		BACKUP PIECE			  26.78 		0					     	7
		IMAGE COPY			      0 			0				   		    0
		FLASHBACK LOG			  3.66 			0	      					3
		FOREIGN ARCHIVED LOG	  0 			0	      					0
		
		--当磁盘空间不足的时候，可以选择手动删除备份（不推荐）
		--通过命令手动删除
		RMAN> delete backup;
	
		using channel ORA_DISK_1
		using channel ORA_DISK_2
	
		List of Backup Pieces
		BP Key  BS Key  Pc# Cp# Status      Device Type Piece Name
		------- ------- --- --- ----------- ----------- ----------
		83      81      1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_12/o1_mf_s_917021157_cr99753k_.bkp
		134     130     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061555_crbv0x1n_.bkp
		135     131     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061555_crbv0wtt_.bkp
		148     145     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T061600_crbv139h_.bkp
		149     146     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T061600_crbv13c3_.bkp
		170     166     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061741_crbv47cy_.bkp
		180     178     1   1   AVAILABLE   DISK        /u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917072267_crbv4cmo_.bkp
	
		Do you really want to delete the above objects (enter YES or NO)? yes
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_12/o1_mf_s_917021157_cr99753k_.bkp RECID=2 STAMP=917021157
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061555_crbv0x1n_.bkp RECID=3 STAMP=917072156
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061555_crbv0wtt_.bkp RECID=4 STAMP=917072156
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T061600_crbv139h_.bkp RECID=5 STAMP=917072163
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T061600_crbv13c3_.bkp RECID=6 STAMP=917072163
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T061741_crbv47cy_.bkp RECID=7 STAMP=917072263
		deleted backup piece
		backup piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917072267_crbv4cmo_.bkp RECID=8 STAMP=917072267
		Deleted 7 objects
	
		--删除备份的拷贝
		RMAN> DELETE COPY OF DATABASE;
	
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=145 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=27 device type=DISK
		specification does not match any datafile copy in the repository
		
		--其他的备份方式
		--将数据库的全被作为拷贝
		--默认备份作为备份集做，除非我们改变RMAN的配置
		--这种备份类型的优点是，在恢复操作是 immediata
		RMAN> BACKUP AS COPY DATABASE PLUS ARCHIVELOG;
	
		Starting backup at 13-JUL-2016 06:32:01
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting archived log copy
		input archived log thread=1 sequence=5 RECID=5 STAMP=917072936
		channel ORA_DISK_2: starting archived log copy
		input archived log thread=1 sequence=6 RECID=2 STAMP=917072152
		output file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_5_crbvz5dt_.arc RECID=8 STAMP=917073126
		channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
		channel ORA_DISK_1: starting archived log copy
		input archived log thread=1 sequence=7 RECID=3 STAMP=917072258
		output file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_6_crbvz5hb_.arc RECID=7 STAMP=917073126
		channel ORA_DISK_2: archived log copy complete, elapsed time: 00:00:01
		channel ORA_DISK_2: starting archived log copy
		input archived log thread=1 sequence=8 RECID=4 STAMP=917072931
		output file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_7_crbvz6wj_.arc RECID=9 STAMP=917073126
		channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
		channel ORA_DISK_1: starting archived log copy
		input archived log thread=1 sequence=9 RECID=6 STAMP=917073121
		output file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_8_crbvz73j_.arc RECID=10 STAMP=917073127
		channel ORA_DISK_2: archived log copy complete, elapsed time: 00:00:00
		output file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_9_crbvz7bt_.arc RECID=11 STAMP=917073127
		channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 06:32:08
	
		Starting backup at 13-JUL-2016 06:32:08
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		channel ORA_DISK_2: starting datafile copy
		input datafile file number=00002 name=/u01/app/oracle/oradata/ocm/sysaux01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_sysaux_crbvzclw_.dbf tag=TAG20160713T063208 RECID=1 STAMP=917073172
		channel ORA_DISK_2: datafile copy complete, elapsed time: 00:00:45
		channel ORA_DISK_2: starting datafile copy
		input datafile file number=00005 name=/u01/app/oracle/oradata/ocm/rcat.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_system_crbvzcmv_.dbf tag=TAG20160713T063208 RECID=2 STAMP=917073177
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:46
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00003 name=/u01/app/oracle/oradata/ocm/undotbs01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_undotbs1_crbw0t4l_.dbf tag=TAG20160713T063208 RECID=3 STAMP=917073186
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:15
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_rcat_crbw0s86_.dbf tag=TAG20160713T063208 RECID=4 STAMP=917073188
		channel ORA_DISK_2: datafile copy complete, elapsed time: 00:00:15
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_users_crbw18y4_.dbf tag=TAG20160713T063208 RECID=5 STAMP=917073193
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:03
		Finished backup at 13-JUL-2016 06:33:15
	
		Starting backup at 13-JUL-2016 06:33:16
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting archived log copy
		input archived log thread=1 sequence=10 RECID=12 STAMP=917073196
		output file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_10_crbw1j6h_.arc RECID=13 STAMP=917073200
		channel ORA_DISK_1: archived log copy complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 06:33:21
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:33:21
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917073203_crbw1mvg_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:33:24

3. 压缩备份，消耗更多的CPU，但节省了大量空间。
	
		--全库备份和归档日志
	
		RMAN> backup as compressed backupset database plus archivelog;
	
		Starting backup at 13-JUL-2016 06:38:56
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting compressed archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=5 RECID=8 STAMP=917073126
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:38:59
		channel ORA_DISK_2: starting compressed archived log backup set
		channel ORA_DISK_2: specifying archived log(s) in backup set
		input archived log thread=1 sequence=6 RECID=7 STAMP=917073126
		input archived log thread=1 sequence=7 RECID=9 STAMP=917073126
		input archived log thread=1 sequence=8 RECID=10 STAMP=917073127
		input archived log thread=1 sequence=9 RECID=11 STAMP=917073127
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 06:38:59
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:39:02
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T063858_crbwd3v6_.bkp tag=TAG20160713T063858 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:03
		channel ORA_DISK_1: starting compressed archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=10 RECID=13 STAMP=917073200
		input archived log thread=1 sequence=11 RECID=14 STAMP=917073536
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:39:03
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 06:39:03
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T063858_crbwd3yl_.bkp tag=TAG20160713T063858 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:04
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:39:04
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T063858_crbwd75p_.bkp tag=TAG20160713T063858 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 06:39:04
	
		Starting backup at 13-JUL-2016 06:39:04
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting compressed full datafile backup set
		channel ORA_DISK_1: specifying datafile(s) in backup set
		input datafile file number=00002 name=/u01/app/oracle/oradata/ocm/sysaux01.dbf
		input datafile file number=00005 name=/u01/app/oracle/oradata/ocm/rcat.dbf
		input datafile file number=00003 name=/u01/app/oracle/oradata/ocm/undotbs01.dbf
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:39:06
		channel ORA_DISK_2: starting compressed full datafile backup set
		channel ORA_DISK_2: specifying datafile(s) in backup set
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 06:39:06
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:39:21
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T063904_crbwdc0f_.bkp tag=TAG20160713T063904 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:15
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 06:39:52
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T063904_crbwdc17_.bkp tag=TAG20160713T063904 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:46
		Finished backup at 13-JUL-2016 06:39:52
	
		Starting backup at 13-JUL-2016 06:39:52
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting compressed archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=12 RECID=15 STAMP=917073592
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:39:56
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:39:57
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_TAG20160713T063954_crbwfwmq_.bkp tag=TAG20160713T063954 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 06:39:57
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:39:57
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917073599_crbwfzp0_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:40:00
	
		--查看占用空间
		SQL> select * from v$recovery_area_usage;
	
		FILE_TYPE	     		PERCENT_SPACE_USED 	PERCENT_SPACE_RECLAIMABLE	NUMBER_OF_FILES
		-------------------- ------------------ -------------------------	---------------
		CONTROL FILE			  0 			0					  		0
		REDO LOG			      0 			0							0
		ARCHIVED LOG			  1.58 		    1.58						3
		BACKUP PIECE			  7.33 		0					     	7
		IMAGE COPY			      0 			0				   		    0
		FLASHBACK LOG			  3.66 			0	      					3
		FOREIGN ARCHIVED LOG	  0 			0	      					0

4. 备份数据库的一部分，比如：tablespace、datafile等等

		--备份一个或多个数据文件
	
		RMAN> backup as copy datafile 1,2;
	
		Starting backup at 13-JUL-2016 06:46:02
		starting full resync of recovery catalog
		full resync complete
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		channel ORA_DISK_2: starting datafile copy
		input datafile file number=00002 name=/u01/app/oracle/oradata/ocm/sysaux01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_system_crbwsj9v_.dbf tag=TAG20160713T064605 RECID=7 STAMP=917073978
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:15
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_sysaux_crbwsjd4_.dbf tag=TAG20160713T064605 RECID=6 STAMP=917073978
		channel ORA_DISK_2: datafile copy complete, elapsed time: 00:00:15
		Finished backup at 13-JUL-2016 06:46:23
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:46:23
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917073985_crbwt1hw_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:46:26
	
		--备份数据文件（datafile）到指定位置
		RMAN> backup as copy datafile 1 format '/u01/app/oracle/flash_recovery_area/OCM/system01.dbf';
	
		Starting backup at 13-JUL-2016 06:48:51
		allocated channel: ORA_DISK_1
		channel ORA_DISK_1: SID=149 device type=DISK
		allocated channel: ORA_DISK_2
		channel ORA_DISK_2: SID=21 device type=DISK
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/system01.dbf tag=TAG20160713T064853 RECID=8 STAMP=917074150
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:25
		Finished backup at 13-JUL-2016 06:49:20
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:49:20
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917074162_crbwzlsd_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:49:23
		--使用备份的默认格式
	
		--http://docs.oracle.com/cd/E11882_01/backup.112/e10643/rcmsubcl010.htm#RCMRF195
	
		RMAN> BACKUP ARCHIVELOG FROM SEQUENCE=1 FORMAT='/u01/app/oracle/flash_recovery_area/OCM/ar_%t_%s_%p' DELETE INPUT;
	
		Starting backup at 13-JUL-2016 06:54:09
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=5 RECID=8 STAMP=917073126
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:54:13
		channel ORA_DISK_2: starting archived log backup set
		channel ORA_DISK_2: specifying archived log(s) in backup set
		input archived log thread=1 sequence=6 RECID=7 STAMP=917073126
		input archived log thread=1 sequence=7 RECID=9 STAMP=917073126
		input archived log thread=1 sequence=8 RECID=10 STAMP=917073127
		input archived log thread=1 sequence=9 RECID=11 STAMP=917073127
		input archived log thread=1 sequence=10 RECID=13 STAMP=917073200
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 06:54:13
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 06:54:14
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/ar_917074453_37_1 tag=TAG20160713T065411 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:01
		channel ORA_DISK_2: deleting archived log(s)
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_6_crbvz5hb_.arc RECID=7 STAMP=917073126
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_7_crbvz6wj_.arc RECID=9 STAMP=917073126
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_8_crbvz73j_.arc RECID=10 STAMP=917073127
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_9_crbvz7bt_.arc RECID=11 STAMP=917073127
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_10_crbw1j6h_.arc RECID=13 STAMP=917073200
		channel ORA_DISK_2: starting archived log backup set
		channel ORA_DISK_2: specifying archived log(s) in backup set
		input archived log thread=1 sequence=11 RECID=14 STAMP=917073536
		input archived log thread=1 sequence=12 RECID=15 STAMP=917073592
		input archived log thread=1 sequence=13 RECID=16 STAMP=917074374
		input archived log thread=1 sequence=14 RECID=17 STAMP=917074449
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 06:54:15
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:54:15
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/ar_917074453_36_1 tag=TAG20160713T065411 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:02
		channel ORA_DISK_1: deleting archived log(s)
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_5_crbvz5dt_.arc RECID=8 STAMP=917073126
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 06:54:16
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/ar_917074455_38_1 tag=TAG20160713T065411 comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:01
		channel ORA_DISK_2: deleting archived log(s)
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_11_crbwd0pt_.arc RECID=14 STAMP=917073536
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_12_crbwfr8c_.arc RECID=15 STAMP=917073592
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_13_crbx66qt_.arc RECID=16 STAMP=917074374
		archived log file name=/u01/app/oracle/flash_recovery_area/OCM/archivelog/2016_07_13/o1_mf_1_14_crbx8k5x_.arc RECID=17 STAMP=917074449
		Finished backup at 13-JUL-2016 06:54:16
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:54:16
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917074459_crbx8vjl_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:54:20
	
		--备份一个或多个表空间
	
		RMAN> backup tablespace users;
	
	
		Starting backup at 13-JUL-2016 06:57:09
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting full datafile backup set
		channel ORA_DISK_1: specifying datafile(s) in backup set
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 06:57:11
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 06:57:12
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnndf_TAG20160713T065709_crbxg7ft_.bkp tag=TAG20160713T065709 comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 06:57:12
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 06:57:12
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917074634_crbxgbxg_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 06:57:15

5. 增量备份策略下数据库的完整备份是0级备份，后续的增量备份被称为1级增量备份，1级增量备份只保存自上次备份级别已更改的块0或1（差异备份）。第二种类型的增量备份是指自上次备份0级备份后复制所有的块（累计备份）。差分备份是为了减少备份的大小和持续时间，而累积类型主要用于以最小化恢复时间。

		--第一次执行生成一个备份作为完全数据库复制（0级因为没有有LEVEL 1）
		
		--第二次执行，执行之后自上次执行后的增量备份（差异）的变化的部分
		--第三次执行和随后的恢复，在数据库的副本制作前做数据库的备份
		--这个脚本有两个优点：
		--1，通过增量备份的备份集小
		--2，从数据文件的精确副本（切换命令）修复（恢复）非常快
		RMAN> 	RUN {
	
		  RECOVER COPY OF DATABASE WITH TAG 'DAILY_BKP_POLICY';
		  BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_BKP_POLICY' DATABASE PLUS ARCHIVELOG;
		}2> 3> 4> 
	
		Starting recover at 13-JUL-2016 07:31:09
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		no copy of datafile 1 found to recover
		no copy of datafile 2 found to recover
		no copy of datafile 3 found to recover
		no copy of datafile 4 found to recover
		no copy of datafile 5 found to recover
		Finished recover at 13-JUL-2016 07:31:10
	
		Starting backup at 13-JUL-2016 07:31:10
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		skipping archived logs of thread 1 from sequence 5 to 10; already backed up
		skipping archived log of thread 1 with sequence 5; already backed up
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=15 RECID=18 STAMP=917076670
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 07:31:15
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 07:31:16
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_DAILY_BKP_POLICY_crbzg3h4_.bkp tag=DAILY_BKP_POLICY comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 07:31:16
	
		Starting backup at 13-JUL-2016 07:31:16
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		no parent backup or copy of datafile 1 found
		no parent backup or copy of datafile 2 found
		no parent backup or copy of datafile 5 found
		no parent backup or copy of datafile 3 found
		no parent backup or copy of datafile 4 found
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		channel ORA_DISK_2: starting datafile copy
		input datafile file number=00002 name=/u01/app/oracle/oradata/ocm/sysaux01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_system_crbzg7dg_.dbf tag=DAILY_BKP_POLICY RECID=10 STAMP=917076706
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:29
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00005 name=/u01/app/oracle/oradata/ocm/rcat.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_sysaux_crbzg7fw_.dbf tag=DAILY_BKP_POLICY RECID=9 STAMP=917076706
		channel ORA_DISK_2: datafile copy complete, elapsed time: 00:00:31
		channel ORA_DISK_2: starting datafile copy
		input datafile file number=00003 name=/u01/app/oracle/oradata/ocm/undotbs01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_rcat_crbzh81x_.dbf tag=DAILY_BKP_POLICY RECID=12 STAMP=917076718
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:08
		channel ORA_DISK_1: starting datafile copy
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_undotbs1_crbzh8f1_.dbf tag=DAILY_BKP_POLICY RECID=11 STAMP=917076718
		channel ORA_DISK_2: datafile copy complete, elapsed time: 00:00:07
		output file name=/u01/app/oracle/flash_recovery_area/OCM/datafile/o1_mf_users_crbzhj4o_.dbf tag=DAILY_BKP_POLICY RECID=13 STAMP=917076720
		channel ORA_DISK_1: datafile copy complete, elapsed time: 00:00:04
		Finished backup at 13-JUL-2016 07:32:02
	
		Starting backup at 13-JUL-2016 07:32:02
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=16 RECID=19 STAMP=917076722
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 07:32:06
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 07:32:07
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_DAILY_BKP_POLICY_crbzhphy_.bkp tag=DAILY_BKP_POLICY comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 07:32:07
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 07:32:07
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917076729_crbzhssp_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 07:32:10
	
		RMAN> RUN {
			  RECOVER COPY OF DATABASE WITH TAG 'DAILY_BKP_POLICY';
			  BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_BKP_POLICY' DATABASE PLUS ARCHIVELOG;
			}2> 3> 4> 
	
		Starting recover at 13-JUL-2016 07:35:56
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		no copy of datafile 1 found to recover
		no copy of datafile 2 found to recover
		no copy of datafile 3 found to recover
		no copy of datafile 4 found to recover
		no copy of datafile 5 found to recover
		Finished recover at 13-JUL-2016 07:35:56
	
	
		Starting backup at 13-JUL-2016 07:35:56
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		skipping archived logs of thread 1 from sequence 5 to 10; already backed up
		skipping archived log of thread 1 with sequence 5; already backed up
		skipping archived logs of thread 1 from sequence 15 to 16; already backed up
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=17 RECID=20 STAMP=917076956
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 07:36:00
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 07:36:01
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_DAILY_BKP_POLICY_crbzq0mt_.bkp tag=DAILY_BKP_POLICY comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 07:36:01
	
		Starting backup at 13-JUL-2016 07:36:01
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting incremental level 1 datafile backup set
		channel ORA_DISK_1: specifying datafile(s) in backup set
		input datafile file number=00001 name=/u01/app/oracle/oradata/ocm/system01.dbf
		input datafile file number=00003 name=/u01/app/oracle/oradata/ocm/undotbs01.dbf
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 07:36:04
		channel ORA_DISK_2: starting incremental level 1 datafile backup set
		channel ORA_DISK_2: specifying datafile(s) in backup set
		input datafile file number=00002 name=/u01/app/oracle/oradata/ocm/sysaux01.dbf
		input datafile file number=00005 name=/u01/app/oracle/oradata/ocm/rcat.dbf
		input datafile file number=00004 name=/u01/app/oracle/oradata/ocm/users01.dbf
		channel ORA_DISK_2: starting piece 1 at 13-JUL-2016 07:36:04
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 07:36:05
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnnd1_DAILY_BKP_POLICY_crbzq4h1_.bkp tag=DAILY_BKP_POLICY comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		channel ORA_DISK_2: finished piece 1 at 13-JUL-2016 07:36:05
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_nnnd1_DAILY_BKP_POLICY_crbzq4lt_.bkp tag=DAILY_BKP_POLICY comment=NONE
		channel ORA_DISK_2: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 07:36:05
	
		Starting backup at 13-JUL-2016 07:36:05
		current log archived
		using channel ORA_DISK_1
		using channel ORA_DISK_2
		channel ORA_DISK_1: starting archived log backup set
		channel ORA_DISK_1: specifying archived log(s) in backup set
		input archived log thread=1 sequence=18 RECID=21 STAMP=917076965
		channel ORA_DISK_1: starting piece 1 at 13-JUL-2016 07:36:09
		channel ORA_DISK_1: finished piece 1 at 13-JUL-2016 07:36:10
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/backupset/2016_07_13/o1_mf_annnn_DAILY_BKP_POLICY_crbzq9tv_.bkp tag=DAILY_BKP_POLICY comment=NONE
		channel ORA_DISK_1: backup set complete, elapsed time: 00:00:01
		Finished backup at 13-JUL-2016 07:36:10
	
		Starting Control File and SPFILE Autobackup at 13-JUL-2016 07:36:10
		piece handle=/u01/app/oracle/flash_recovery_area/OCM/autobackup/2016_07_13/o1_mf_s_917076972_crbzqf17_.bkp comment=NONE
		Finished Control File and SPFILE Autobackup at 13-JUL-2016 07:36:14
	
		RMAN> 
	
	
		--我们可以使用相同的脚本执行类型的累积增量备份
		RUN {
			 RECOVER COPY OF DATABASE WITH TAG 'DAILY_BKP_POLICY';
			 BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_BKP_POLICY' CUMULATIVE DATABASE PLUS ARCHIVELOG;
		}

6.  最常用的check/list命令
	
		CROSSCHECK BACKUP;
		CROSSCHECK ARCHIVELOG ALL;
		BACKUP VALIDATE DATABASE;            # physical corruptions (Check V$DATABASE_BLOCK_CORRUPTION)
		CHECK LOGICAL BACKUP DATABASE;       # Logical Corruptions (. Info in Alert and session trace file)
		RESTORE DATABASE VALIDATE;
		REPORT NEED BACKUP;
		REPORT OBSOLETE;
		REPORT UNRECOVERABLE;
		REPORT NEED BACKUP INCREMENTAL 2;    # Datafiles that require more than 2 INC in Recovery
		REPORT NEED BACKUP REDUNDANCY 2;
		REPORT NEED BACKUP RECOVERY WINDOW OF 7 DAYS;
		REPORT SCHEMA;
		REPORT SCHEMA AT TIME 'SYSDATE-7';
		VALIDATE DATABASE PLUS ARCHIVELOG;
		VALIDATE CHECK LOGICAL TABLESPACE USERS;
		VALIDATE CHECK LOGICAL DATAFILE 4;
		VALIDATE CHECK LOGICAL DATAFILE 4 BLOCK 1 TO 10;
		VALIDATE ARCHIVELOG ALL;
		VALIDATE CHECK LOGICAL CURRENT CONTROLFILE;
		VALIDATE CHECK LOGICAL BACKUPSET 6446;
		RESTORE DATABASE PREVIEW;
		RECOVER DATABASE PREVIEW;
		LIST BACKUP OF DATABASE;
		LIST BACKUP OF ARCHIVELOG FROM SCN 2953216;
		LIST BACKUPPIECE '/u01/app/oracle/fast_recovery_area/OCM/autobackup/2013_06_07/o1_mf_s_817462899_8v31bmrs_.bkp';
		LIST BACKUPPIECE 8222;
		LIST BACKUPSET 8220;
		LIST COPY OF TABLESPACE SYSTEM;
		LIST COPY OF DATABASE ARCHIVELOG FROM TIME='SYSDATE-7';
		LIST INCARNATION;
		LIST EXPIRED BACKUP;
		LIST BACKUP SUMMARY;                		# TY = B/P (Backupset/Proxy copy)
											# LV = [01]/F/A (Incremental level X/Datafiles/Archivelogs)
											# S = A/U/X (Available/Unavailable/Expired)

7. 查看备份集以及数据的一些视图
	 
		V$ARCHIVED_LOG
		V$COPY_CORRUPTION
		V$DATABASE_BLOCK_CORRUPTION
		V$RMAN_CONFIGURATION
		V$BACKUP_<CORRUPTION|SET|SPFILE|DATAFILE|DEVICE|FILES|PIECE|REDOLOG|SYNC_IO|ASYNC_IO>
		V$PROXY_<ARCHIVEDOG|DATAFILE>

8. 当Backup/Restore执行超过6秒的时候，可以查看V$SESSION_LONGOPS来查询任务的状态

		ALTER SESSION SET NLS_DATE_FORMAT='YYYY/MM/DD HH24:MI:SS';
		COL OPNAME FORMAT A30
		SELECT
		  SID,
		  SERIAL#,
		  OPNAME,
		  SOFAR,
		  TOTALWORK,
		  ROUND(SOFAR/TOTALWORK*100,2) COMPLETE,
		  TO_DATE(SYSDATE+(TIME_REMAINING/(24*60*60))) as ESTTIME
		from
		  V$SESSION_LONGOPS
		WHERE
		  TOTALWORK <> 0
		  AND SOFAR <> TOTALWORK
		ORDER BY 1