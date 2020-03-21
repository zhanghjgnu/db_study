$ rman target /
RMAN>show all;
RMAN>show channel;

RMAN> show all;

RMAN configuration parameters for database with db_unique_name CDBRAC are:
CONFIGURE RETENTION POLICY TO REDUNDANCY 5;
CONFIGURE BACKUP OPTIMIZATION OFF; # default
CONFIGURE DEFAULT DEVICE TYPE TO DISK; # default
CONFIGURE CONTROLFILE AUTOBACKUP ON; # default
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; # default
CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
CONFIGURE MAXSETSIZE TO UNLIMITED; # default
CONFIGURE ENCRYPTION FOR DATABASE OFF; # default
CONFIGURE ENCRYPTION ALGORITHM 'AES128'; # default
CONFIGURE COMPRESSION ALGORITHM 'BASIC' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE ; # default
CONFIGURE RMAN OUTPUT TO KEEP FOR 7 DAYS; # default
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default
CONFIGURE SNAPSHOT CONTROLFILE NAME TO '/u01/app/oracle/product/12.2.0.1/db_1/dbs/snapcf_cdbrac1.f'; # default

RMAN> show channel;

RMAN configuration parameters for database with db_unique_name CDBRAC are:
RMAN configuration has no stored or default parameters


    在oracle12c中rman备份的路径应该是这样的优先级:
    备份语句中指定的format  > rman 中显现的configure channel device type disk format '/oracle/orclarch/%U_%d'的路径 > 闪回恢复区>$ORACLE_HOME/dbs  
    如果rman没有显现的配置备份路径，也就是没有如下操作，
RMAN>configure channel device type disk format '/oracle/orclarch/%U_%d';  


BACKUP DATABASE FORMAT '/oracle_arch/arch2/%d_db_%u';
通过配置将备份文件存储到指定路径，如 /oracle_arch/arch2
configure channel device type disk format '/oracle_arch/arch2/%d_db_%u';
后面的%d_db_%u是存储格式

Starting Control File and SPFILE Autobackup at 21-MAR-20
piece handle=/u01/app/oracle/product/12.2.0.1/db_1/dbs/c-519686032-20200321-00 comment=NONE
Finished Control File and SPFILE Autobackup at 21-MAR-20


在RMAN里
RMAN>crosscheck backup;   →查出无效的备份
RMAN>delete noprompt expired backup;  →删除无效的备份
RMAN> report obsolete ; →删除后再查下还是否有过期的备份：
