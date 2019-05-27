# Mysql存储过程

```sql
-- 设置结束符为// ，因为在命令行中sql的结束符为';' 而在存储过程中会出现';'，可能会使命令行混淆。
delimiter //
DROP PROCEDURE if EXISTS `test_procedure`
CREATE PROCEDURE `test_procedure` (
IN inVal CHAR,
OUT outVal CHAR
)
BEGIN
	SELECT ef FROM test WHERE abc=inVal INTO outVal;
END //
```

其中 IN关键字表示输入参数，

OUT关键字表示输出返回参数。



调用方式为

```
call test_procedure(1,@TestParam);
select @TestParam;
```





http://wanghf0218.iteye.com/blog/260667