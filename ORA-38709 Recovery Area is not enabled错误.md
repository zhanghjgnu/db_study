## ORA-38709: Recovery Area is not enabled错误
运行ALTER DATABASE FLASHBACK ON;的时候提示错误:
```
ERROR at line 1:
ORA-38706: Cannot turn on FLASHBACK DATABASE logging.
ORA-38709: Recovery Area is not enabled.
```
我们在安装Oracle数据库软件时，有个选项是flash recovery area，如果我们没有勾选，就表示不启用快速恢复区域，我们可以在数据库装好之后开启或关闭。
开启或关闭FRA需要数据库在mount状态下并且开启归档模式，执行alter database flashback on/off;
### 解决办法:
oracle是需要我们去设置DB_RECOVERY_FILE_DEST参数，这个代表FRA的存储路径，设置DB_RECOVERY_FILE_DEST这个参数前必须先设置DB_RECOVERY_FILE_DEST_SIZE，这个是FRA空间大小

1. 设置参数
```
 alter system set db_recovery_file_dest_size=5G;
 alter system set db_recovery_file_dest='/u01/app/oracle/fras';
```
2. 设置数据库FRA为on
```
  alter database flashback on;
```
