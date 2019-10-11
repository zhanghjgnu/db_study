    oracle调试打印变量用dbms_output.putline();pl/sql developer单步调试功能也很完善。
     pg函数和存储过程可以用raise来实现变量打印，pgAdmin有debugger扩展，不过修改修改postgresql.conf 文件
```
shared_preload_libraries = ‘$libdir/other_libraries/plugin_debugger’
```
    修改后重启数据库。因为使用pg函数默认是一个事务，函数执行完毕提交，失败全部回滚，所以没必要把pg调试搞得那么复杂(pg11开始在存储过程中支持事务了，和oracle越来越像),直接在pgadmin中显示中间变量执行情况就够用。

raise语法
```
RAISE [ level ] 'format' [, expression [, ... ]] [ USING option = expression [, ... ] ];
RAISE [ level ] condition_name [ USING option = expression [, ... ] ];
RAISE [ level ] SQLSTATE 'sqlstate' [ USING option = expression [, ... ] ];
RAISE [ level ] USING option = expression [, ... ];
RAISE ;
```

  其中 level 可以选择DEBUG, LOG, INFO, NOTICE, WARNING, 和 EXCEPTION, 缺省是EXCEPTION 。
format是信息格式，格式字符串指定要报告的错误消息文本。格式字符串后面可以是要插入到消息中的可选参数表达式。在格式字符串中，%替换为下一个可选参数值的字符串表示形式。写入%%以发出文本%。参数的数目必须与格式字符串中的%placeholders的数目相匹配，否则在编译函数期间会引发错误。
例如
```
RAISE NOTICE 'Calling cs_create_job(%)', v_job_id;
raise notice ‘this is raise demo , param1 is % ,param2 is %’,param1,param2;
```
通过使用后跟option=表达式项进行写入，可以将其他信息附加到错误报告。每个表达式可以是任何字符串值表达式。允许的选项关键字是
MESSAGE 错误消息文本
DETAIL 错误详细信息消息。
HINT 提示信息
ERRCODE
COLUMN
CONSTRAINT
DATATYPE
TABLE
SCHEMA

举例
```
RAISE EXCEPTION 'Nonexistent ID --> %', user_id
      USING HINT = 'Please check your user ID';
```
