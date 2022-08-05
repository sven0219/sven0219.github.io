---
title: Oralce 数据库修改实例名和数据库名
tags:
  - Oracle
  - 数据库
  - 运维
id: '12500'
categories:
  - - skills
date: 2021-10-13 17:01:04
---

# Oralce 数据库修改实例名和数据库名

分为两个阶段，第一阶段修改实例名sid；第二阶段修改数据库名dbname 将原先的实例名PSPROD 更改为 PSUAT 将原先的数据库名PSPROD 更改为 PSUAT

## 1\. 修改实例名 sid

### 1.1 查看当前实例名

```bash
# 登录数据库
su - oracle
sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Wed Oct 13 15:53:09 2021

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

# 查看实例名
SQL> select instance from v$thread;

INSTANCE
--------------------------------------------------------------------------------
PSPROD



```

### 1.2 关闭数据库

```sql
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SQL> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
```

### 1.3 修改/etc/oratab文件

```bash
# 将文件里面的PSPROD换成PSUAT
vim /etc/oratab
```

### 1.4 修改.bash\_profile文件

编辑oracle用户的 .bash\_profile文件,将文件里面的PSPROD换成PSUAT

```bash
vim ~/.bash_profile
source ~/.bash_profile
# 查看是否生效
envgrep ORACLE
```

### 1.5 修改dbs目录下的文件名

dbs目录是用于存放数据库服务器端的参数文件Spfile、初始化文件init、还有密码文件orapw$ORACLE\_SID

```bash
cd $ORACLE_HOME/dbs
mv hc_PSPROD.dat  hc_PSUAT.dat
mv lkPSPROD lkPSUAT
mv spfilePSPROD.ora  spfilePSUAT.ora
```

重新生成密码文件，并将旧的密码文件删除

```bash
orapwd file=$ORACLE_HOME/dbs/oraw$ORACLE_SID password=Admin123. entries=5 force=y
rm -rf orapwPSPROD
```

### 1.6 登录数据库查看

```bash
$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Wed Oct 13 16:05:24 2021

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup
ORACLE instance started.

Total System Global Area 8053063680 bytes
Fixed Size          8639712 bytes
Variable Size        1593838368 bytes
Database Buffers     6442450944 bytes
Redo Buffers            8134656 bytes
Database mounted.
Database opened.
SQL> select instance from v$thread;

INSTANCE
--------------------------------------------------------------------------------
PSUAT

```

## 2\. 修改数据库名dbname

不用退出登录，接着开始修改数据库名dbname

### 2.1 备份控制文件，并关闭退出数据库

```sql
SQL> alter database backup controlfile to trace resetlogs;

Database altered.

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
```

### 2.2 根据旧的控制文件生成修改更新控制文件的sql语句

进入控制文件的备份目录，根据alter\_orcl.log找到控制备份文件

```bash
/data/app/oracle/diag/rdbms/psprod/PSUAT/trace
vim alert_PSUAT.log
```

在 alert\_PSUAT.log 文件中找到contolfile的备份trc Backup controlfile written to trace file /data/app/oracle/diag/rdbms/psprodstb5/PSUAT/trace/PSUAT\_ora\_3619.trc 复制一份进行修改：

```bash
cp /data/app/oracle/diag/rdbms/psprodstb5/PSUAT/trace/PSUAT_ora_3619.trc /tmp/PSUAT.sql
```

编辑文件isdms.sql 1）查找STARTUP NOMOUNT语句，将这一行上面的所有行都删除 2）将-- End of tempfile additions.语句下面的行全部删除 3）查找所有以–开始的行，把这些行删除 4）查找所有的PSPROD修改为PSUAT，所有的psprod修改为psuat 5）找到CREATE CONTROLFILE REUSE DATABASE…语句，将其中的REUSE修改为SET 6）找到RECOVER DATABASE USING BACKUP CONTROLFILE语句，将其用双横线（–）注释掉 修改完后：

```sql
STARTUP NOMOUNT
CREATE CONTROLFILE SET DATABASE "PSUAT" RESETLOGS FORCE LOGGING ARCHIVELOG
    MAXLOGFILES 16
    MAXLOGMEMBERS 3
    MAXDATAFILES 200
    MAXINSTANCES 8
    MAXLOGHISTORY 292
LOGFILE
  GROUP 4 '/data/app/oracle/oradata/PSUAT/redo04.log'  SIZE 500M BLOCKSIZE 512,
  GROUP 5 '/data/app/oracle/oradata/PSUAT/redo05.log'  SIZE 500M BLOCKSIZE 512,
  GROUP 6 '/data/app/oracle/oradata/PSUAT/redo06.log'  SIZE 500M BLOCKSIZE 512
-- STANDBY LOGFILE
--   GROUP 1 '/data/app/oracle/oradata/PSUAT/standby01.log'  SIZE 500M BLOCKSIZE 512,
--   GROUP 2 '/data/app/oracle/oradata/PSUAT/standby02.log'  SIZE 500M BLOCKSIZE 512,
--   GROUP 3 '/data/app/oracle/oradata/PSUAT/standby03.log'  SIZE 500M BLOCKSIZE 512,
--   GROUP 7 '/data/app/oracle/oradata/PSUAT/standby04.log'  SIZE 500M BLOCKSIZE 512
DATAFILE
  '/data/app/oracle/oradata/PSUAT/system01.dbf',
  '/data/app/oracle/oradata/PSUAT/aaapp.dbf',
  '/data/app/oracle/oradata/PSUAT/sysaux01.dbf',
  '/data/app/oracle/oradata/PSUAT/undotbs01.dbf',
  '/data/app/oracle/oradata/PSUAT/psdefault.dbf',
  '/data/app/oracle/oradata/PSUAT/users01.dbf',
  '/data/app/oracle/oradata/PSUAT/aalarge.dbf',
  '/data/app/oracle/oradata/PSUAT/adapp.dbf',
  '/data/app/oracle/oradata/PSUAT/amapp.dbf',
  '/data/app/oracle/oradata/PSUAT/avapp.dbf',
  '/data/app/oracle/oradata/PSUAT/bdapp.dbf',
  '/data/app/oracle/oradata/PSUAT/bnapp.dbf',
  '/data/app/oracle/oradata/PSUAT/bnlarge.dbf',
  '/data/app/oracle/oradata/PSUAT/ccapp.dbf',
  '/data/app/oracle/oradata/PSUAT/coapp.dbf',
  '/data/app/oracle/oradata/PSUAT/cuaudit.dbf',
  '/data/app/oracle/oradata/PSUAT/cularg1.dbf',
  '/data/app/oracle/oradata/PSUAT/cularg2.dbf',
  '/data/app/oracle/oradata/PSUAT/cularg3.dbf',
  '/data/app/oracle/oradata/PSUAT/cularge.dbf',
  '/data/app/oracle/oradata/PSUAT/diapp.dbf',
  '/data/app/oracle/oradata/PSUAT/dtapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eobfapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eocfapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eocmapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eocmlrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eocmwrk.dbf',
  '/data/app/oracle/oradata/PSUAT/eocuapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoculrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eodsapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eodslrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eoecapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoeclrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eoecwrk.dbf',
  '/data/app/oracle/oradata/PSUAT/eoeiapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoeilrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eoewapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoewlrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eoewwrk.dbf',
  '/data/app/oracle/oradata/PSUAT/eoiuapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoiulrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eoiuwrk.dbf',
  '/data/app/oracle/oradata/PSUAT/eolarge.dbf',
  '/data/app/oracle/oradata/PSUAT/eoltapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eoppapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eopplrg.dbf',
  '/data/app/oracle/oradata/PSUAT/eotpapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eotplrg.dbf',
  '/data/app/oracle/oradata/PSUAT/epapp.dbf',
  '/data/app/oracle/oradata/PSUAT/eplarge.dbf',
  '/data/app/oracle/oradata/PSUAT/erapp.dbf',
  '/data/app/oracle/oradata/PSUAT/erlarge.dbf',
  '/data/app/oracle/oradata/PSUAT/erwork.dbf',
  '/data/app/oracle/oradata/PSUAT/faapp.dbf',
  '/data/app/oracle/oradata/PSUAT/falarge.dbf',
  '/data/app/oracle/oradata/PSUAT/fgapp.dbf',
  '/data/app/oracle/oradata/PSUAT/fglarge.dbf',
  '/data/app/oracle/oradata/PSUAT/fsapp.dbf',
  '/data/app/oracle/oradata/PSUAT/giapp.dbf',
  '/data/app/oracle/oradata/PSUAT/gpapp.dbf',
  '/data/app/oracle/oradata/PSUAT/gpdeapp.dbf',
  '/data/app/oracle/oradata/PSUAT/hpapp.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp1.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp2.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp3.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp4.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp5.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp6.dbf',
  '/data/app/oracle/oradata/PSUAT/hrapp7.dbf',
  '/data/app/oracle/oradata/PSUAT/hrimage.dbf',
  '/data/app/oracle/oradata/PSUAT/hrlarg1.dbf',
  '/data/app/oracle/oradata/PSUAT/hrlarge.dbf',
  '/data/app/oracle/oradata/PSUAT/hrsapp.dbf',
  '/data/app/oracle/oradata/PSUAT/hrsarch.dbf',
  '/data/app/oracle/oradata/PSUAT/hrslarge.dbf',
  '/data/app/oracle/oradata/PSUAT/hrswork.dbf',
  '/data/app/oracle/oradata/PSUAT/hrwork.dbf',
  '/data/app/oracle/oradata/PSUAT/htapp.dbf',
  '/data/app/oracle/oradata/PSUAT/inapp.dbf',
  '/data/app/oracle/oradata/PSUAT/paapp.dbf',
  '/data/app/oracle/oradata/PSUAT/palarge.dbf',
  '/data/app/oracle/oradata/PSUAT/pcapp.dbf',
  '/data/app/oracle/oradata/PSUAT/pclarge.dbf',
  '/data/app/oracle/oradata/PSUAT/piapp.dbf',
  '/data/app/oracle/oradata/PSUAT/pilarge.dbf',
  '/data/app/oracle/oradata/PSUAT/piwork.dbf',
  '/data/app/oracle/oradata/PSUAT/poapp.dbf',
  '/data/app/oracle/oradata/PSUAT/psimage.dbf',
  '/data/app/oracle/oradata/PSUAT/psimage2.dbf',
  '/data/app/oracle/oradata/PSUAT/psimgr.dbf',
  '/data/app/oracle/oradata/PSUAT/psindex.dbf',
  '/data/app/oracle/oradata/PSUAT/ptamsg.dbf',
  '/data/app/oracle/oradata/PSUAT/ptapp.dbf',
  '/data/app/oracle/oradata/PSUAT/ptappe.dbf',
  '/data/app/oracle/oradata/PSUAT/ptaudit.dbf',
  '/data/app/oracle/oradata/PSUAT/ptcmstar.dbf',
  '/data/app/oracle/oradata/PSUAT/ptlock.dbf',
  '/data/app/oracle/oradata/PSUAT/ptprc.dbf',
  '/data/app/oracle/oradata/PSUAT/ptprjwk.dbf',
  '/data/app/oracle/oradata/PSUAT/ptrpts.dbf',
  '/data/app/oracle/oradata/PSUAT/psmatvw.dbf',
  '/data/app/oracle/oradata/PSUAT/pttbl.dbf',
  '/data/app/oracle/oradata/PSUAT/pttlrg.dbf',
  '/data/app/oracle/oradata/PSUAT/pttree.dbf',
  '/data/app/oracle/oradata/PSUAT/ptwork.dbf',
  '/data/app/oracle/oradata/PSUAT/pvapp.dbf',
  '/data/app/oracle/oradata/PSUAT/py0lrg.dbf',
  '/data/app/oracle/oradata/PSUAT/pyapp.dbf',
  '/data/app/oracle/oradata/PSUAT/pylarge.dbf',
  '/data/app/oracle/oradata/PSUAT/pywork.dbf',
  '/data/app/oracle/oradata/PSUAT/saapp.dbf',
  '/data/app/oracle/oradata/PSUAT/sacapp.dbf',
  '/data/app/oracle/oradata/PSUAT/salarge.dbf',
  '/data/app/oracle/oradata/PSUAT/srapp.dbf',
  '/data/app/oracle/oradata/PSUAT/stapp.dbf',
  '/data/app/oracle/oradata/PSUAT/stlarge.dbf',
  '/data/app/oracle/oradata/PSUAT/stwork.dbf',
  '/data/app/oracle/oradata/PSUAT/tlapp.dbf',
  '/data/app/oracle/oradata/PSUAT/tllarge.dbf',
  '/data/app/oracle/oradata/PSUAT/tlwork.dbf',
  '/data/app/oracle/oradata/PSUAT/waapp.dbf'
CHARACTER SET AL32UTF8
;
-- RECOVER DATABASE USING BACKUP CONTROLFILE
ALTER DATABASE OPEN RESETLOGS;
ALTER TABLESPACE TEMP ADD TEMPFILE '/data/app/oracle/oradata/PSUAT/temp01.dbf'
     SIZE 306184192  REUSE AUTOEXTEND ON NEXT 655360  MAXSIZE 32767M;
ALTER TABLESPACE PSTEMP ADD TEMPFILE '/data/app/oracle/oradata/PSUAT/pstemp01.dbf'
     SIZE 314572800  REUSE AUTOEXTEND OFF;
ALTER TABLESPACE PSGTT01 ADD TEMPFILE '/data/app/oracle/oradata/PSUAT/psgtt01.dbf'
     SIZE 524288000  REUSE AUTOEXTEND ON NEXT 5242880  MAXSIZE 32767M;
```

### 2.3 生成配置文件

```
sqlplus  / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Wed Oct 13 16:19:11 2021

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> create pfile   from spfile;

File created.

SQL> exit
Disconnected
```

最终生成的文件在$ORACLE\_HOME/dbs目录下文件名为init$ORACLE\_SID.ora 在本示例中为：initPSUAT.ora

### 2.4 目录和文件修改

#### 2.4.1 initPSUAT.ora文件修改

这个文件是上面生成的配置文件 1）将PSPROD.开头的配置项删除 2）将文件中的PSPROD替换成PSUAT，psprod替换成psuat

#### 2.4.2 init.ora文件修改

将文件中的PSPROD替换成PSUAT，psprod替换成psuat

#### 2.4.3 spfilePSUAT.ora文件修改

1）将PSPROD.开头的配置项删除 2）将文件中的PSPROD替换成PSUAT，psprod替换成psuat

#### 2.4.4 删除文件lkPSPROD

```bash
rm lkPSPROD
```

### 2.5 修改$ORACLE\_BASE/admin目录下的目录和文件内容

#### 2.5.1 将admin目录下的PSPROD目录名重命名为PSUAT

```bash
cd $ORACLE_BASE/admin
mv PSPROD PSUAT
```

### 2.6 修改$ORACLE\_BASE/diag目录下的目录

###### 修改目录名，删除旧的psprod目录

```bash
cd $ORACLE_BASE/diag/rdbms
mv psprod psuat
cd psuat
rm -rf PSPROD
```

### 2.7 修改$ORACLE\_BASE/oradata目录下的目录和文件内容

```bash
cd $ORACLE_BASE/oradata
mv PSPROD/ PSUAT
# 删除控制文件
cd PSUAT
rm -rf control01.ctl  
```

### 2.8 修改监听的配置文件tnsnames.ora

```bash
cd $ORACLE_HOME/network/admin
vim tnsnames.ora 
# 修改 host 及 PSPROD 改为 PSUAT
```

### 2.8 调用前面生成的 sql 生成控制文件

```sql
 sqlplus / as sysdba
SQL> @/tmp/PSUAT.sql
ORA-01081: cannot start already-running ORACLE - shut it down first

Control file created.


Database altered.


Tablespace altered.


Tablespace altered.


Tablespace altered.

## 查看是否修改成功

SQL> select open_mode from v$database;

OPEN_MODE
------------------------------------------------------------
READ WRITE

SQL> show parameter name

NAME                     TYPE
------------------------------------ ---------------------------------
VALUE
------------------------------
cdb_cluster_name             string
PSUAT
cell_offloadgroup_name           string

db_file_name_convert             string
/data/app/oracle/oradata/PSUAT
, /data/app/oracle/oradata/PSU
AT
db_name                  string

NAME                     TYPE
------------------------------------ ---------------------------------
VALUE
------------------------------
PSUAT
db_unique_name               string
PSUAT
global_names                 boolean
FALSE
instance_name                string
PSUAT
lock_name_space              string


NAME                     TYPE
------------------------------------ ---------------------------------
VALUE
------------------------------
log_file_name_convert            string

pdb_file_name_convert            string

processor_group_name             string

service_names                string
PSUAT
```

## 3\. 重启数据库及监听

重启数据库

```sql
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area 8053063680 bytes
Fixed Size          8639712 bytes
Variable Size        1593838368 bytes
Database Buffers     6442450944 bytes
Redo Buffers            8134656 bytes
Database mounted.
Database opened.
SQL> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
```

重启监听

```bash
lsnrctl start
```