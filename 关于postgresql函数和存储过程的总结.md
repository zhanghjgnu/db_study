# postgresql函数与存储过程
###### postgresql版本10以前只有function,没有procedure,从版本11开始才有了procedure,总体感觉用pl/pgsql编程越来越像oracle的pl/sql.
存储过程和函数（Function）类似，不过它没有返回值。存储过程最大的优势就是能够支持事务控制，也就是可以在定义中使用 COMMIT 
或者 ROLLBACK 语句编程了。使用 CREATE\ALTER\DROP PROCEDURE 命令创建\修改\删除存储过程，使用 CALL 命令调用存储过程.
当然postgresql后台的函数和存储过程还有很多语言实现，除了PL/pgSQL还有PL/Perl、PL/Python、PL/Tcl 以及 SPI ,不过后台直接操作数据库当然还是PL/pgSQL最顺手了。
  # 使用PL/pgSQL的优点
  SQL被PostgreSQL和大多数其他关系数据库用作查询语言。它是可移植的并且容易学习。但是每一个SQL语句必须由数据库服务器单独执行。这意味着客户端应用必须发送每一个查询到数据库服务器、等待它被处理、接收并处理结果、做一些计算，然后发送更多查询给服务器。如果客户端和数据库服务器不在同一台机器上，所有这些会引起进程间通信并且将带来网络负担。
  1. 通过PL/pgSQL，将一整块计算和一系列查询分组在数据库服务器内部，这样就有了一种过程语言的能力并且使 SQL 更易用，节省了相当多的客户端/服务器通信开销。
  2. 消除客户端和服务器之间的额外往返通信被
  3. 客户端不需要的中间结果不必被整理或者在服务器和客户端之间传送
  4. 避免多轮的查询解析 
  5. 还有，通过PL/pgSQL你可以使用 SQL 中所有的数据类型、操作符和函数。

   # 函数的语法
``` 
CREATE FUNCTION somefunc(integer, text) RETURNS integer
AS 'function body text'
LANGUAGE plpgsql;
```
    就CREATE FUNCTION而言，函数体只是一个字符串文字。使用美元引用编写函数体， 而不是普通的单引号语法通常很有帮助。没有美元引用， 函数体中的任何单引号或反斜杠都必须通过加倍来转义。
函数内部
```
[ <<label>> ]
[ DECLARE
    declarations ]
BEGIN
    statements
END [ label ];  
```

``` 
CREATE FUNCTION somefunc() RETURNS integer AS $$
<< outerblock >>
DECLARE
    quantity integer := 30;
BEGIN
    RAISE NOTICE 'Quantity here is %', quantity;  -- Prints 30
    quantity := 50;
    --
    -- 创建一个子块
    --
    DECLARE
        quantity integer := 80;
    BEGIN
        RAISE NOTICE 'Quantity here is %', quantity;  -- Prints 80
        RAISE NOTICE 'Outer quantity here is %', outerblock.quantity;  -- Prints 50
    END;

    RAISE NOTICE 'Quantity here is %', quantity;  -- Prints 50

    RETURN quantity;
END;
$$ LANGUAGE plpgsql;
```

函数调用使用select

存储过程例子，存储过程没有返回值,不过参数可以用OUT来返回，而且可以有多个OUT参数(函数也可以的)
```
CREATE PROCEDURE transaction_test2()
LANGUAGE plpgsql
AS $$
DECLARE
    r RECORD;
BEGIN
    FOR r IN SELECT * FROM test2 ORDER BY x LOOP
        INSERT INTO test1 (a) VALUES (r.x);
        COMMIT;
    END LOOP;
END;
$$;

CALL transaction_test2();
```

存储过程调用语法
CALL name ( [ argument ] [, ...] )
存储过程调用比较简单，使用 CALL 命令即可，而函数的调用是使用 SELECT 命令。
