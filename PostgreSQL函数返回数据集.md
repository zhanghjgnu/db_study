## 第一种：声明setof 某表/某视图 返回类型 
建表
```
create table employee(id int primary key, name text, salary int, departmentid int references department);
(1)
create or replace function f_get_employee() 
returns setof employee 
as 
$$
select * from employee;
$$
language 'sql';
(2)
create or replace function f_get_employee_query() 
returns setof employee 
as 
$$
begin
return query select * from employee;
end;
$$
language plpgsql;
```
查询出来的函数还可以像普通的表一样按条件查询

## 第二种:声明setof record返回类型 
用处在于“一定程度上”解决了动态返回结果集字段的问题
用新建type来构造返回的结果集 
```
create type dept_salary as (departmentid int, totalsalary int);

create or replace function f_dept_salary() 
returns setof dept_salary 
as
$$
declare
rec dept_salary%rowtype;
begin
for rec in select departmentid, sum(salary) as totalsalary from f_get_employee() group by departmentid loop
  return next rec;
  end loop;
return;
end;
$$
language 'plpgsql';
```
## 第三种:根据执行函数变量不同返回不同数据集
```
create or replace function f_get_rows(text) returns setof record as
$$
declare
rec record;
begin
for rec in EXECUTE 'select * from ' || $1 loop
return next rec;
end loop;
return;
end
$$
language 'plpgsql';
```
执行函数
 select * from f_get_rows('department') as dept(deptid int, deptname text);
或
select * from f_get_rows('employee') as employee(employee_id int, employee_name text,employee_salary int,dept_id int);
这样一个函数可以返回不同的结果

## 第四种：声明refcursor游标）返回类型 
```
CREATE OR REPLACE FUNCTION function1 () RETURNS refcursor AS 
$body$ 
DECLARE 
    result refcursor; 
BEGIN 
   open result for select * from table1,table2; --你可以任意选择你想要返回的表和字段 
return result; 
END; 
$body$ 
LANGUAGE 'plpgsql'
```
推荐使用第三种
