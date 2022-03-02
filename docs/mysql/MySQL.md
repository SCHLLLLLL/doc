## MySQL命令

创建表

```sql
create table 表名{
	列名 列类型,
	列名 列类型,
	...
};
```

查看表结构

```sql
desc 表名;
```

查看服务器版本

```sql
select version()
mysql --version
mysql --V
```

### 查询

*查询顺序：表-->条件-->字段-->排序*

####  基础查询

```sql
SELECT 查询列表 FROM 表名
```

**查询列表：可以是 `字段` `常量值` `表达式` `函数`**

**查询的结果是一个虚拟的表格**

**对字段可以使用着重号==`==用以区分关键字 eg:** 

```sql
SELECT `NAME` FROM t_text;
```

**查询常量**

```sql
SELECT 100;
SELECT 'Lin';
```

**查询表达式**

```SQL
SELECT 100*99;
SELECT 100%99;
```

**查询函数**

```SQL
SELECT VERSION();
```

**查询_起别名**（提高阅读性、区分重名）

关键字：`AS`

```SQL
#方式一
SELECT 100%99 AS 别名;
SELECT last_name AS 姓,first_name AS 名 FROM t_user;
#方式二
SELECT last_name 姓,first_name 名 FROM t_user;
```

对别名可以使用双引号==“ ”==区分关键字 eg：

```sql
#OUT为关键字
SELECT salary AS "out put" FROM employee;
```

**查询_去重**

关键字：`DISTINCT`

```sql
SELECT DISTINCT department_id FROM employee;
```

#### 排序查询

关键字：`ORDER BY`

```SQL
SELECT 查询列表 FROM 表名 [WHERE 筛选条件] ORDER BY 排序列表;
```

排序列表：单个字段，多个字段，表达式，函数，列数，别名，以上及以上组合

升序： `ASC`  默认

降序： `DESC`

```SQL
#字段排序
SELECT * FROM employee WHERE salary>1000 ORDER BY salary DESC;
#多字段爬排序 先按工资升序，再按部门编号降序
SELECT * FROM employee WHERE salary>1000 ORDER BY salary ASC,department_id DESC;
#别名排序
SELECT last_name,salary*12*(1+IFNULL(comssion_pct,0)) AS 年薪 FROM employee ORDER BY 年薪;
#函数排序
SELECT last_name FROM employee ORDER BY LENGTH(last_name);
#列数排序(第几列，类似单个字段)
SELECT * FROM 2;
```



#### 条件查询

```sql
SELECT 查询列表 FROM 表名 WHERE 筛选条件;
```

分类：

- 按条件表达式筛选

  条件运算符：`<` `>` `=` `<>(!=)` `>=` `<=`

  `=` 无法判断NULL

- 按逻辑表达式筛选

  逻辑运算符：`&&` `||` `!` 推荐使用`AND` `OR` `NOT`

- 模糊查询

  `LIKE` `BETWEEN` `IN` `IS NULL`

**用于在 WHERE 子句中搜索列中的指定模式**

**关键字：**`LIKE`

```sql
#查询名字中包含a的员工信息
SELECT * FROM employee WHERE last_name LIKE '%a%';
SELECT * FROM employee WHERE last_name LIKE '_a';
```

通配符：

- `%`  不确定的字符（包含0个字符）
- `_`  不确定的单个字符

转义字符：' \ '

`\_`  `\%`

```sql
#关键字：ESCAPE  标识那个符号为转义字符
SELECT * FROM employee WHERE last_name LIKE '_$_a'; ESCAPE '$';
```

**关键字：**`BETWEENT AND` (包含临界值)

`BETWEEN AND` 相当于  两个`OR`

```SQL
#条件不能颠倒
SELECT * FROM employee WHERE salary BETWEEN 1000 AND 2000;
```

**关键字：**`IN`

`IN ` 相当于 `=`

判断字段的值是否属于IN列表中的某一项，列表中的值类型必须一致或兼容

```SQL
SELECT * FROM employee WHERE job_id IN('IT_PROT','AD_VP','AD_PRES');
```

**关键字：**`IS NULL`

```sql
SELECT * FROM employee WHERE salary IS NULL;
SELECT * FROM employee WHERE salary IS NOT NULL;
```

**安全等于**  `<=>`

可以判断NULL

```sql
SELECT * FROM employee WHERE salary <=>NULL;
```























**"+"号的作用****

只作为运算符

```;
SELECT 100+90;
SELECT '123'+90;
#将字符转换成数值型，无法转换则当做0计算
SELECT 'Lin'+90;
SELECT 'Lin'+'HangHui';
#若有null，结果为null
SELECT null+10;
```

### 函数(常见函数)

#### 字符函数

**拼接字符**  `CONCAT()`

若拼接的字段内容为NULL，则会拼接结果该字段会全为NULL

```SQL
SELECT CONCAT(last_name,first_name) AS 姓名 FROM t_user;
#若有字段为NULL，则该字段的结果为全为空
SELECT CONCAT(last_name,first_name,',',age) AS 结果 FROM t_user;
#------------------
SELECT CONCAT(last_name,first_name,',',IFNULL(age,0)) AS 结果 FROM t_user;
```

**获取字节长度**  `LENGTH()`

```sql
SELECT LENGTH('HELLO 你好');
```

**获取字符长度**  `CHAR_LENGTH()`

```SQL
SELECT CHAR_LENGTH('HELLO WORLD');
```

**截取字符串**  `SUBSTRING()` 

```SQL
起始索引从1开始
#参数：(字符,起始索引,截取长度)
SELECT SUBSTR('你好世界',1,2); #你好
#参数：(字符,起始索引)
SELECT SUBSTR('你好世界',3); #世界
```

**截取左边或右边字符**  `LEFT()`  / `RIGHT()`

```sql
SELECT LEFT('你好世界',1); #你
SELECT RIGTH('你好世界',2); #世界
```

**获取字符第一次出现的索引**  `INSTR()`

```sql
SELECT INSTR('你好世界你好世界你好世界','世界');
```

**去掉前后指定字符**  `TRIM()`

```SQL
#默认去掉空格
SELECT TRIM('   你好   世界');
SELECT TRIM('x' FROM 'xxxx你好xxx世界xxxx');
```

**左填充/右填充**  `LPAD()`/ `RPAD()`

```SQL
SELECT LPAD('你好',10,'世界'); #世界世界世界世界你好
SELECT RPAD('你好',10,'世界'); #你好世界世界世界世界
SELECT RPAD('你好',1,'世界'); #你
```

**字符大小写**  `UPPER()`  / `LOWER()`

```SQL
SELECT UPPER('a');
SELECT LOWER('B');
```

**比较两个字符大小**  `STRCMP()`

```SQL
SELECT STRCMP('abc','aaa');#返回1,0或者-1
```

#### 数学函数

**绝对值 ** `ABS()`

```SQL
SELECT ABS(-2.3);
```

**向上取整**  `CEIL()`

```SQL
SELECT CELL(1.02);
```

**向下取整**  `FLOOR()`

```SQL
SELECT FLOOR(-1.02);
```

**四舍五入**  `ROUND()`

```SQL
SELECT ROUND(1.8782);
SELECT ROUND(1.8782,2);
```

**保留小数点后几位（截断）**  `TRUNCATE()`

```SQL
SELECT TRUNCATE(1.887223,2);
```

**取余**  `MOD()`

```SQL
SELECT MOD(-10,3);
```

**求和**  `SUM()`

```SQL
SELECT SUM(salary) FROM employee;
```

**平均数** `AVG()`

```SQL
SELECT AVG(salary) FROM employee;
```

**最大值** `MAX()`

```sql
SELECT MAX(salary) FROM employee;
```

**最小值** `MIN()`

```sql
SELECT MIN(salary) FROM employee;
```

**计算非空字段的个数** `COUNT()`

```sql
SELECT COUNT(*) FROM employee;
SELECT COUNT(salary) FROM employee;
SELECT COUNT(DISTINCT department_id) FROM employee;
```

#### 日期函数

**获取当前时间**  `NOW()`

```SQL
SELECT NOW();
```

**获取当前日期**  `CURDATE()`

```SQL
SELECT CURDATE();
```

**获取当前时间**   `CURTIME()`

```SQL
SELECT CURTIME();
```

**获取两个时间相差的天数**  `DATEDIFF()`

```SQL
SELECT DATEDIFF('2020-01-22','1998-03-17');
```

**日期格式转换**  `DATE_FORMAT()`

```SQL
SELECT DATE_FORMAT('1998-03-17','%Y年%m月%d%日 %H时%m分%s秒') '时间';
```

![image-20200122210542252](C:\Users\42516\Desktop\笔记\MySQL\images\image-20200122210542252.png)

**字符串转日期格式**  `STR_TO_DATE()`

```SQL
SELECT STR_TO_DATE('03/17 1998','%m/%d %Y');
SELECT * FROM USER WHERE birthday<STR_TO_DATE('03/17 1998','%m/%d %Y');
```

#### 流程控制函数

**判断**  `IF()`

```SQL
SELECT IF(100>99,'GOOD','BAD');
```

**分支**

```SQL
CASE 表达式
WHEN 值 THEN 结果
WHEN 值 THEN 结果
ELSE 结果
END
```

```SQL
SELECT std_id,grade,
CASE
WHEN grade>60 THEN 'C'
WHEN grade>80 THEN 'B'
WHEN grade>90 THEN 'A'
ELSE 'D'
END '等级'
FROM tb_student;
```

**判断是否为空**

关键字：`IFNULL()`

```SQL
SELECT IFNULL(age,0) AS 年龄;
```



### 规范

- 不区分大小写，但建议关键字大写，表名、列名小写
- 每条命令用分号结尾
- 每条命令根据需要，进行缩进或者换行
- 注释
  - 单行注释 #注释文字
  - 单行注释 -- 注释文字
  - 多行注释  /* 注释文字 * /
- 