## 关于UUID的生成方法
药品主键采用UUID来实现,我们有三个选项来生成 UUID，

###1.在应用程序代码中
  可以使用golang来实现
  ```
package	                                            id	                                                              format
https://github.com/segmentio/ksuid 266	0pPKHjWprnVxGH7dEsAoXX2YQvU	4 bytes of time (seconds) + 16 random bytes
https://github.com/rs/xid 546	        b50vl5e54p1000fo3gh0	4 bytes of time (seconds) + 3 byte machine id + 2 byte process id + 3 bytes random
https://github.com/kjk/betterguid 59	-Kmdih_fs4ZZccpx2Hl1	8 bytes of time (milliseconds) + 9 random bytes
https://github.com/sony/sonyflake 147	20f8707d6000108	~6 bytes of time (10 ms) + 1 byte sequence + 2 bytes machine id
https://github.com/oklog/ulid 93	01BJMVNPBBZC3E36FJTGVF0C4S	6 bytes of time (milliseconds) + 8 bytes random
https://github.com/chilts/sid 65	1JADkqpWxPx-4qaWY47~FqI	8 bytes of time (ns) + 8 random bytes
https://github.com/satori/go.uuid 386	5b52d72c-82b3-4f8e-beb5-437a974842c	UUIDv4 from http://tools.ietf.org/html/rfc4122 33 for comparison
```
### 2.在pg数据库中使用 uuid-ossp 扩展
 create extension "uuid-ossp" 是安装 uuid_generate_v4() 扩展函数；
   select uuid_generate_v4() 是检验函数，下面是生成的结果。

### 3.在pg数据库中使用 pgcrypto 扩展
CREATE EXTENSION pgcrypto;
使用下面的 SQL 语句，就可以生成一个 UUID
SELECT gen_random_uuid();
使用 pgcrypto 中的 gen_random_uuid() 对表在磁盘上的键空间碎片有负面影响。Random产生非常片段的插入，这会破坏表。使用 uuid_generate_v1mc() [代替]…键是seq，因为它们是基于时间的。所以所有插入都指向同一个数据页，没有随机 io。
这是有道理的，由于随机概率分布的键，它应该是碎片。但是，这种碎片对数据库系统本身的效率不是很好。为了获得较低的使用 UUID 的好处主键用于分裂也许指出了最好使用 uuid_generate_v1mc() 
