---
layout: post
title:  "Create the Database"
date:   2016-07-25 07:48:57 +0000
comments: true
categories: Server_Configuration
---


1. oracle 官方文档 -> 2 Day DBA -> (2) Installing Oracle Database and Creating a Database -> Creating and Managing a Database with DBCA

2. 登录Oracle用户

3. 执行dbca来创建数据库

4. 进入到下一个屏幕 -> 点击 “Next”

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