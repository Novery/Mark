## 语法
+ mvl空判断
+ decode 类似if else
+ merge into
```sql
MERGE INTO A USING B ON (A.条件字段 = B.条件字段)
when MATCH THEN UPDATE SET A.待更新字段 = B.更新字段
when NOT MATCHED THEN
INSERT (A.字段) VALUES （B.字段）

```
+ union:对两个结果集进行并集操作，不包括重复行，排序
+ union all：并集操作，包括重复行，不进行排序
+ intersect：交易操作，不包括重复行，排序
+ minus：差操作，不包括重复行，排序

## 事务
+ 脏读：一个事务读取其他事务写入但没提交的数据
+ 不可重复读：一个事务再次读取其之前曾经读取过的数据，发现数据被删除或修改
+ 幻读：一个事务再次读取之前的数据，发现数据被其他事务插入，结果集不同
+ 未提交读取（read uncommitted）脏、重复、幻
+ 读已提交：解决脏读
+ 可重复读取（repeatable read）：解决脏读、重复读
+ 串行化（rerializable）:解决幻读

## SQL优化
+ 删除重复记录
```sql
DELETE FROM EMP E WHERE E.ROWID > (SELECT MIN(X.ROWID) FROM EMP X WHERE X.EMP_NO = E.EMP_NO);
```
+ 用TRUNCATE替代DELETE
```
当删除表记录时，回滚段用来存放可以被恢复的数据，如果不提交，可以用来回滚，当使用truncate时，回滚段将不存放可回滚的信息，没有数据能恢复，速度也会更快了，只用于删除全表数据；
同理，减少回滚段的数据能提高效率，多加commit
```
+ 当数据量小，delete快于truncate，因为truncate其实是先drop后建表的
```
（mysql官方文档）
Truncate operations drop and re-create the table, which is much faster than deleting rows one by one, particularly for large tables
截断操作删除并重新创建表，这比逐个删除行快得多，尤其是对于大型表
```

+ oracle的解析自下而上，需要把过滤掉最大数量记录的条件必须写在WHERE子句的末尾
+ 尽量避免where子句中使用！=或<>操作符，否则将不走索引
+ 避免is null,不走索引，可设置默认值0
+ 避免where子句中使用or，不走索引，可使用unionall
+ 前置百分号，like '%
+ in not in,不走索引，使用exist
+ 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算
+ 如果某列大量重复数据，如性别，则建立索引也不会生效
+ exist替代in
+ 尽量使用数字型字段,引擎在处理查询和连接的时候，对于字符串会逐一比对，数字型只比较一次
+ varchar替代char，节省空间
+ 避免select *，返回了多余的字段
+ 避免大事务操作，提高系统并发能力


