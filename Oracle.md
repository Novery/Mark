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
+ 不可重复度：一个事务再次读取其之前曾经读取过的数据，发现数据被删除或修改
+ 幻读：一个事务再次读取之前的数据，发现数据被其他事务插入，结果集不同
 

现象|脏读取|不可重复读取|不存在读取  
-|-|- 
香蕉 | $1 | 5 |
苹果 | $1 | 6 |
草莓 | $1 | 7 |
