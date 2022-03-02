**查询第二高薪水，如果不存在第二高薪水，那么查询返回null**

解题思路：通过排序后limit得到第二高薪水

limit n,m：跳过n条数据取出第m条数据。

```sql
SELECT IFNULL((SELECT distinct Salary from Employee order by Sarlary DESC limit 1,1),NULL);
```

**查询第n高薪水，如果不存在第二高薪水，那么查询返回null**

limit n offset m：跳过m条数据取n条。

```sql
CREATE FUNCTION getNthHightestSalary(N INT) RETURNS INT
BEGIN
	SET N = N - 1;
	RETURN (
    	SELECT IFNULL((SELECT DISTINCT Salary FROM Employee order by Sarlary DESC 			LIMIT 1 OFFSET N));
    );
END
# 这里 （LIMIT 1 OFFSET N ）也可以写成 （LIMIT N,1）
```

**分数排名**

解题思路：计算每个分数比他大的个数，将这个个数作为Rank

```sql
SELECT Score,(SELECT count(distinct Score) FROM Scores WHERE Score >= c.Score) Rank
FROM Scores c ORDER BY DESC;
```

