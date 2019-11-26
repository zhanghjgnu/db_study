## postgresql导入txt数据文件总结

### 建立国家和地区词典
```
create table Dict_country(
country_id varchar(6),
country_cn varchar(100) not null primary key,
country_en varchar(100),
tel_code varchar(10),
time_diff numeric(6,2))
```

### 原始数据2.txt
```
Angola,安哥拉,AO,244,-7
Afghanistan,阿富汗,AF,93,0
Albania,阿尔巴尼亚,AL,355,-7
Algeria,阿尔及利亚,DZ,213,-8
Andorra,安道尔共和国,AD,376,-8
Anguilla,安圭拉岛,AI,1264,-12
Antigua and Barbuda,安提瓜和巴布达,AG,1268,-12
Argentina,阿根廷,AR,54,-11
Armenia,亚美尼亚,AM,374,-6
Ascension,阿森松, ,247,-8
Australia,澳大利亚,AU,61,+2
Austria,奥地利,AT,43,-7
Azerbaijan,阿塞拜疆,AZ,994,-5
Bahamas,巴哈马,BS,1242,-13
Bahrain,巴林,BH,973,-5
Bangladesh,孟加拉国,BD,880,-2
Barbados,巴巴多斯,BB,1246,-12
Belarus,白俄罗斯,BY,375,-6
Belgium,比利时,BE,32,-7
Belize,伯利兹,BZ,501,-14
Benin,贝宁,BJ,229,-7
Bermuda Is.,百慕大群岛,BM,1441,-12
Bolivia,玻利维亚,BO,591,-12
Botswana,博茨瓦纳,BW,267,-6
Brazil,巴西,BR,55,-11
Brunei,文莱,BN,673,0
Bulgaria,保加利亚,BG,359,-6
Burkina-faso,布基纳法索,BF,226,-8
Burma,缅甸,MM,95,-1.3
Burundi,布隆迪,BI,257,-6
Cameroon,喀麦隆,CM,237,-7
Canada,加拿大,CA,1,-13
Cayman Is.,开曼群岛, ,1345,-13
Central African Republic,中非共和国,CF,236,-7
Chad,乍得,TD,235,-7
Chile,智利,CL,56,-13
China,中国,CN,86,0
Colombia,哥伦比亚,CO,57,0
 ... ...
```

### 先给eastwill_emr2开通超级用户权限
```
ALTER USER eastwill_emr2 WITH SUPERUSER;
```
然后执行导入操作
```
copy Dict_country(country_en ,country_cn ,country_id ,tel_code ,time_diff ) from 'd:/2.txt' DELIMITER ','
```
结果提示
编码"GBK"的字符 0xxx0xxx在编码"UTF8"没有相对应值

### 查看服务器和本地的字符集
```
postgres=# show server_encoding;
 server_encoding
-----------------
 UTF8
postgres=# show client_encoding;
 client_encoding
-----------------
 GBK
 ```
果然不一样，修改本地字符集
```
postgres=# set client_encoding to 'utf8';
SET
```
再执行
```
copy Dict_country(country_en ,country_cn ,country_id ,tel_code ,time_diff ) from 'd:/2.txt' DELIMITER ','
```
成功

copy导入的语法
```
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | STDIN }
    [ [ WITH ] ( option [, ...] ) ]

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
    TO { 'filename' | STDOUT }
    [ [ WITH ] ( option [, ...] ) ]

where option can be one of:

    FORMAT format_name
    OIDS [ boolean ]
    DELIMITER 'delimiter_character'
    NULL 'null_string'
    HEADER [ boolean ]
    QUOTE 'quote_character'
    ESCAPE 'escape_character'
    FORCE_QUOTE { ( column_name [, ...] ) | * }
    FORCE_NOT_NULL ( column_name [, ...] )
    ENCODING 'encoding_name'
```
### 今天导入一个疾病编码词典，一直提示invalid byte sequence for encoding "UTF8": 0xb9
忽然想到有ENCODING选项，采用如下命令导入成功(不能换行):
```
copy clover_mir.diag_benxi_ln(icd_id, icd_name, begtime, endtime, pinyin_code, wubi_code, valid_flag, injury_flag, birth_flag, diag_cate, diag_type, diag_spec, opcode, optime, remark, diag_type_degree) from '/var/lib/postgresql/data/icd10_bx_ln.txt' ENCODING 'GBK';
```
