## postgresql函数返回数据集的总结

postgresql自定义函数可以返回refcursor/record,也可以返回自定义type，直接返回table。其中返回的refcursor/record最好在pg内部调用，其他语言比如golang调用后不方便处理。
返回自定义type和返回table差不多，下面是一个返回table的例子
```    
CREATE OR REPLACE FUNCTION clover_odr.sch_reg4cash_list(thsp_code text)
    RETURNS TABLE(pid1 text, patient_name1 text, patient_type1 text, reg_type1 text,pay_should1 numeric(12,4))
    LANGUAGE 'plpgsql'
AS $cloveropen$
    -- 查询待交款的患者列表
   -- 返回列表 门诊号 姓名 挂号类别 用户类别 待交款合计
DECLARE
    tpid clover_odr.out_reg.pid%type;
    tpatient_name clover_odr.out_reg.patient_name%type;	
    tpatient_type clover_odr.out_reg.patient_type%type;
    treg_type clover_odr.out_reg.reg_type%type;
    tpay_should clover_odr.out_reg.pay_cash%type := 0.00;	
    cursor_reg4cash refcursor;
BEGIN
    execute 'DROP TABLE IF EXISTS t_reg4cash';
    execute 'CREATE TEMP TABLE t_reg4cash(pid text, patient_name text, patient_type text, reg_type text,pay_should numeric(12,4))';
	
    OPEN cursor_reg4cash FOR
      SELECT distinct pid,patient_name, patient_type, reg_type
      FROM clover_odr.fee_detail where hsp_code=thsp_code and presc_status='9' order by patient_name;
        FETCH  cursor_reg4cash into tpid,tpatient_name, tpatient_type, treg_type;
    while found loop
        SELECT coalesce(sum(real_price*quantity*days),0.00) into tpay_should
           FROM clover_odr.fee_detail
           where pid=tpid and patient_name=tpatient_name and patient_type=tpatient_type and reg_type=treg_type;	
      insert into t_reg4cash values (tpid,tpatient_name, tpatient_type, treg_type, tpay_should);
      FETCH next from  cursor_reg4cash into tpid,tpatient_name, tpatient_type, treg_type;	  
    end loop;
    close cursor_reg4cash; 
    RETURN QUERY select pid, patient_name, patient_type, reg_type,pay_should from t_reg4cash order by patient_name;	      
END;
$cloveropen$;
```
这样返回结果是括号括起来用逗号分隔的字段记录,其他语言比较容易处理，不过还是不够方便，还有更好的解决办法吗，答案是有的。
这就是返回json array结果集再转换为字符串,golang对json的处理很方便,下面是改进的代码:
```
CREATE OR REPLACE FUNCTION clover_odr.sch_reg4cash_list_new(thsp_code text)
    RETURNS text
    LANGUAGE 'plpgsql'
AS $cloveropen$
    -- 查询待交款的患者列表
    -- 返回列表 门诊号 姓名 挂号类别 用户类别 待交款合计
DECLARE
    tpid clover_odr.out_reg.pid%type;
    tpatient_name clover_odr.out_reg.patient_name%type;	
    tpatient_type clover_odr.out_reg.patient_type%type;
    treg_type clover_odr.out_reg.reg_type%type;
    tpay_should clover_odr.out_reg.pay_cash%type := 0.00;	
    cursor_reg4cash refcursor;
    tout_str text;
BEGIN
    execute 'DROP TABLE IF EXISTS t_reg4cash';
    execute 'CREATE TEMP TABLE t_reg4cash(pid text, patient_name text, patient_type text, reg_type text,pay_should numeric(12,4))';
	
     OPEN cursor_reg4cash FOR
        SELECT distinct pid,patient_name, patient_type, reg_type
	  FROM clover_odr.fee_detail where hsp_code=thsp_code and presc_status='9' order by patient_name;
	FETCH  cursor_reg4cash into tpid,tpatient_name, tpatient_type, treg_type;
    while found loop
        SELECT coalesce(sum(real_price*quantity*days),0.00) into tpay_should
          FROM clover_odr.fee_detail
        where pid=tpid and patient_name=tpatient_name and patient_type=tpatient_type and reg_type=treg_type;
        insert into t_reg4cash values (tpid,tpatient_name, tpatient_type, treg_type, tpay_should);
        FETCH next from  cursor_reg4cash into tpid,tpatient_name, tpatient_type, treg_type;	  
    end loop;
    close cursor_reg4cash; 
    select array_to_json(array_agg(row_to_json(t)))::text into tout_str
    from (
      select pid, patient_name, patient_type, reg_type,pay_should from t_reg4cash
    ) AS t;

    RETURN tout_str;	      
END;
$cloveropen$;
```
返回结果：
[{"pid":"000000029001","patient_name":"白龙马","patient_type":"01","reg_type":"ptmz","pay_should":12345678.1200},{"pid":"000000029001","patient_name":"白龙马","patient_type":"01","reg_type":"tbmz","pay_should":4.6800}]
这样的json array使用go或者js来处理都非常方便，多语言编程的接口应该给不同语言调用提供最大方便。
