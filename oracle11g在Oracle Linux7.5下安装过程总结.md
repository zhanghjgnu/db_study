## oracle11g在Oracle Linux 7.5下安装过程总结

最近一直用docker,很少有机会安装裸机的oracle数据库了，今天为了恢复一台服务器数据不得已去安装一次。
环境:  操作系统Oracle Linux 7.5 数据库 oracle11.2.0.1 X64,安装后升级到oracle11.2.0.3
按照https://oracle-base.com/articles/11g/oracle-db-11gr2-installation-on-oracle-linux-7介绍的文档采用自动安装步骤安装:
### 1 修改/etc/hosts
```
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.0.215   ol7.localdomain  ol7
```
###2 修改/etc/hostname
```
ol7.localdomain
```
### 3 安装支持文件,这个时间有点儿长，不过好在不用逐个去安装那些包
```
# yum install oracle-rdbms-server-11gR2-preinstall
# yum update
```
### 4 修改oracle密码
```
passwd oracle
```
### 5 设置防火墙
 修改 /etc/selinux/config，改为 SELINUX=permissive
 ```
# setenforce Permissive
# systemctl stop firewalld
# systemctl disable firewalld
```
### 6 建立安装路径
```
mkdir -p /u01/app/oracle/product/11.2.0/db_1
chown -R oracle:oinstall /u01
chmod -R 775 /u01
```
### 7 修改oracle用户的环境变量
用oracle登录，修改/home/oracle/.bash_profile文件
```
# Oracle Settings
TMP=/tmp; export TMP
TMPDIR=$TMP; export TMPDIR

ORACLE_HOSTNAME=ol7.localdomain; export ORACLE_HOSTNAME
ORACLE_UNQNAME=oracle; export ORACLE_UNQNAME
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0.4/db_1; export ORACLE_HOME
ORACLE_SID=oracle; export ORACLE_SID
ORACLE_TERM=xterm; export ORACLE_TERM
PATH=/usr/sbin:$PATH; export PATH
PATH=$ORACLE_HOME/bin:$PATH; export PATH

LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH
```
### 8 用oracle登录进入图形界面,开一个terminal
   ./runInstaller &
如果正常就会检查安装需求然后安装了，不过过程不是那么顺利
### 9 安装过程中遇到 error invoking the "ins_emagent.mk" file 错误
解决办法编辑 "$ORACLE_HOME/sysman/lib/ins_emagent.mk"文件，修改如下:
```
FROM:
$(MK_EMAGENT_NMECTL)
TO  :
$(MK_EMAGENT_NMECTL) -lnnz11
```
### 10 然后遇到ins_ctx.mk错误，这个参考https://stackoverflow.com/questions/28774848/oracle-database-installation-issue-in-ins-ctx-mk解决
就是在root用户下建立一个repair_oracle.sh
```
# Fix ctx/lib/ins_ctx.mk

ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1

cat << __EOF__ > /tmp/memcpy_wrap.c
#include <stddef.h>
#include <string.h>

asm (".symver wrap_memcpy, memcpy@GLIBC_2.14");
void *wrap_memcpy(void *dest, const void *src, size_t n) {
return memcpy(dest, src, n);
}
__EOF__

if [[ -e "${ORACLE_HOME}/ctx/lib/ins_ctx.mk" ]]; then
sed -i -e 's/\$(INSO_LINK)/\$(INSO_LINK) -Wl,--wrap=memcpy_wrap \$(ORACLE_HOME)\/ctx\/lib\/memcpy_wrap.o/g' ${ORACLE_HOME}/ctx/lib/ins_ctx.mk
gcc -c /tmp/memcpy_wrap.c -o ${ORACLE_HOME}/ctx/lib/memcpy_wrap.o && rm /tmp/memcpy_wrap.c
fi
```
然后chmod +x,执行这个文件

### 11 用p10404530_112030_Linux-x86-64的两个补丁去升级
  解压后得到database目录，oracle下运行./runInstaller -jreLoc /etc/alternatives/jre_1.8.0
启动runInstaller时指定为本地jdk，否则中间弹出对话框显示不全，不能完整显示提示信息，以及按钮。
 安装目录选择 /u01/app/oracle/product/11.2.0.3/db_1,选择升级数据库
### 12 安装过程中又提示Error in invoking target 'agent nmhs' of makefile错误，解决办法
修改$ORACLE_HOME/sysman/lib/ins_emagent.mk，将
$(MK_EMAGENT_NMECTL)修改为：$(MK_EMAGENT_NMECTL) -lnnz11
注意这个$ORACLE_HOME=/u01/app/oracle/product/11.2.0.3/db_1，不是升级前的
经过漫长的等待，ok，完活了！

