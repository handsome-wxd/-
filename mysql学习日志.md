数据库范式：

第一范式：列不可再分

第二范式：一张表只只表达一层含义（只描述一件事情）

第三范式：表中每一列和主键之间的是直接依赖关系，而不是间接依赖

[toc]



# 1.SQL语言的分类

- DDL（数据分类语言）：CREATE\ALTER\DROP\RENAME\TRUNCATE
- DML（数据操作语言）：INSERT\DELETE\UPDATE\SELECT
- DCL（数据控制语言）:COMMIT\ROLLBACK\SAVEPOINT\GRANK\REVOKE

# 2.别名，去重复，筛选,限制结果

```mysql
SELECT first_name fn,last_name AS LN,email "e"
FROM employees;#别名三种方式
#去重复
SELECT DISTINCT department_id 
FROM employees;
#表名和关键字一样 加``进行区分
SELECT * FROM `order`；
#显示表的字段的详细信息
DESCRIBE employees;
#筛选信息用WHERE关键字,例如筛选first_name
SELECT * FROM employees
WHERE first_name='Steven';
#限制结果 表示从第二行开始，取五个数
SELECT department_id 
FROM employees
limit 1,5;
```

# 3.运算符

```mysql
#算数运算符

#+ - * 、 % 数字字符串转为数字，不是数字的为0
SELECT 100,100+50,100-50,100*50,100/50,100%50,100+'1',100+'a'
FROM DUAL;

#比较运算符
#查询部门id不为10的员工信息
SELECT *
from employess
where department_id <>10;
#等价于
SELECT *
from employess
where department_id !=10;

#等式两边都为非数字字符串，则通过NSI进行操作
SELECT 'a'='b',0='a' FROM DUAL;

#只要有null 参与结果就为null
SELECT NULL=NULL FROM DUAL;

#<=> 安全等于，两边都为null返回1 一边为null 返回0,
SELECT NULL<=>NULL,1<=>NULL FROM DUAL;

#is null\is not null\isnull
SELECT last_name,salary,commission_pct FROM employees
WHERE commission_pct IS NULL;

SELECT last_name,salary,commission_pct FROM employees
WHERE commission_pct IS NOT NULL;

SELECT last_name,salary,commission_pct FROM employees
WHERE ISNULL(commission_pct);

#LEAST() \GREATEST  返回最小 \最大的
SELECT LEAST('G','B','T','M'),GREATEST('G','B','T','M')
FROM DUAL;

SELECT LEAST(first_name,last_name),GREATEST(LENGTH(first_name),LENGTH(last_name))
FROM employees;

#Beteen and
SELECT last_name,salary,commission_pct FROM employees
WHERE salary BETWEEN 6000 AND 8000 AND commission_pct IS NOT NULL ;

#in ,not in
SELECT last_name,salary,commission_pct,department_id FROM employees
WHERE department_id IN(10,20,30) ;

SELECT last_name,salary,commission_pct,department_id FROM employees
WHERE department_id NOT IN(10,20,30) ;

#LIke:模糊查询 查找last_name包括'a'的
# %匹配一个或者多个元素  _匹配一个元素
SELECT last_name FROM employees
WHERE last_name LIKE '%a%';

#以'a'开头
SELECT last_name FROM employees
WHERE last_name LIKE 'a%';

#以'a'结尾
SELECT last_name FROM employees
WHERE last_name LIKE '%a';

#同时包含 'a'和'k'
SELECT last_name FROM employees
WHERE last_name LIKE '%a%' AND '%k%';

#第二个字符是'a'的 '_'代表不确定字符
SELECT last_name FROM employees
WHERE last_name LIKE '_a%';

#第二个字符是'_'的 第三个是'a'
SELECT last_name FROM employees
WHERE last_name LIKE '_\_a%';

#REGEXP\RLIKE 正则表达式,以s开头的
SELECT 'shkerr' REGEXP '^S' FROM DUAL;

```

# 4.REGEXP  正则表达式

```mysql
#匹配所有包括 10的 例如110 101
SELECT * 
FROM employees
WHERE employee_id REGEXP '10'
#like 只匹配是10的
SELECT * 
FROM employees
WHERE employee_id LIKE '10'

#or  匹配所有包括 10 和11的
SELECT * 
FROM employees
WHERE employee_id REGEXP '10|11'

#匹配多个选项  匹配包含 1 或 2 或 3
SELECT * 
FROM employees
WHERE employee_id REGEXP '[1,2,3]'

#范围匹配
#匹配包含 1 或者 2 .。。 或者 9
SELECT * 
FROM employees
WHERE employee_id REGEXP '[1-9]'

#转义字符\\
#匹配包含.的
SELECT * 
FROM employees
WHERE employee_id REGEXP '\\.'
```

![image-20220912222814441](C:\Users\18440\AppData\Roaming\Typora\typora-user-images\image-20220912222814441.png)

![image-20220912222846013](C:\Users\18440\AppData\Roaming\Typora\typora-user-images\image-20220912222846013.png)

![image-20220912222907276](C:\Users\18440\AppData\Roaming\Typora\typora-user-images\image-20220912222907276.png)

## LIKE和REGEXP的区别

- **在匹配内容上的区别** 
  LIKE要求整个数据都要匹配，用Like，必须这个字段的所有内容满足条件；

REGEXP只需要部分匹配即可，只需要有任何一个片段满足即可。

- **在匹配位置上的区别**

LIKE 匹配整个列，如果被匹配的文本在列值中出现，LIKE 将不会找到它，相应的行也不会被返回（除非使用通配符）；

REGEXP 在列值内进行匹配，如果被匹配的文本在列值中出现，REGEXP 将会找到它，相应的行将被返回，并且 REGEXP 能匹配整个列值（与 LIKE 相同的作用）。

- **SQL语句返回数据区别**

LIKE匹配 ：该SQL语句将不返回数据；

REGEXP匹配 ：该SQL语句会返回一行数据；

- **速度区别**



# 5.排序和分页

```mysql
#当同时使用where和order by时，order by 应该位于where后

#选择按照年终奖进行排序，排序方式DESC为降序，ASC为升序
SELECT employee_id,last_name,salary*12 annual
FROM employees
ORDER BY annual DESC;

SELECT employee_id,last_name
FROM employees
WHERE department_id IN (70,60,50)
ORDER BY department_id DESC;
#二级排序，先按照department_id进行降序排列，如果department_id相同时,然后根据salary进行升序排列
SELECT employee_id,last_name,department_id
FROM employees
WHERE department_id IN (70,60,50)
ORDER BY department_id DESC, salary ASC;

#分页 //从第一个数据开始，一页20个数据
SELECT employee_id,last_name,department_id
FROM employees
LIMIT 0,20;
#8.0新特性
SELECT employee_id,last_name,department_id
FROM employees
LIMIT 20 OFFSET 2;

```

5.多表查询

![image-20220129183644810](C:\Users\18440\AppData\Roaming\Typora\typora-user-images\image-20220129183644810.png)

```mysql
#中图内连接
SELECT last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id`=d.`department_id`;

#左上图
SELECT last_name,department_name
FROM employees e LEFT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`;

#右上图
SELECT last_name,department_name
FROM employees e RIGHT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`;
#左中图
SELECT last_name,department_name
FROM employees e RIGHT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`
WHERE d.`department_id` IS NULL;
#右中图
SELECT last_name,department_name
FROM employees e RIGHT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`
WHERE e.`department_id` IS NULL;
#左下图 全连接图，左上+右中
SELECT last_name,department_name
FROM employees e LEFT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`
UNION ALL
SELECT last_name,department_name
FROM employees e RIGHT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`
WHERE e.`department_id` IS NULL;
#右下图 左中+右中
SELECT last_name,department_name
FROM employees e RIGHT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`
WHERE d.`department_id` IS NULL
UNION ALL
SELECT last_name,department_name
FROM employees e RIGHT OUTER JOIN departments d
ON e.`department_id`=d.`department_id`
WHERE e.`department_id` IS NULL;

```

自然连接，自动寻找两个表中相同的字段，进行等值连接

```mysql
#自然连接
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id`=d.`department_id`
AND e.`manager_id`=d.`manager_id`;
#与上面效果相同
SELECT employee_id,last_name,department_name
FROM employees e NATURAL JOIN departments d;
```

using

```mysql
SELECT last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id`=d.`department_id`;
#相同
SELECT last_name,department_name
FROM employees e JOIN departments d
USING (department_id);

```

6.流程控制函数

```mysql
#流程控制
#IF(value,value1,value2)
SELECT last_name,salary,IF(salary>=6000,'高工资','低工资')"details"
FROM employees;
#case when
SELECT last_name,salary,CASE WHEN salary>=15000 THEN '白骨精'
			     WHEN salary>=15000 THEN '屌丝故'
			     WHEN salary>=15000 THEN '小屌丝'
			     ELSE '草根' END "detail"
FROM employees;
```

7.group by

```mysql
#group by
#根据计算每个部门的平均id
SELECT department_name,employees.department_id,AVG(salary)
FROM employees JOIN departments d
GROUP BY department_id
#查询各个department_id,job_id的平均工资
SELECT department_id,job_id,AVG(salary)
FROM employees
GROUP BY department_id,job_id;
#如果条件使用了聚合函数，则必使用having来替换where。否则报错
#查询各个部门中最高工资大于10000部门
SELECT department_id,MAX(salary)
FROM employees
GROUP BY department_id
HAVING	MAX(salary)>10000;
#查询部门id为10，20，30，40，这四个部门中最高工资比10000高的部门
#方式一
SELECT department_id,MAX(salary)
FROM employees
GROUP BY department_id
HAVING	MAX(salary)>10000 AND department_id IN (10,20,30,40);

SELECT department_id,MAX(salary)
FROM employees
WHERE department_id IN (10,20,30,40)
GROUP BY department_id
HAVING	MAX(salary)>10000 ;
```

8.子查询

```mysql
#子查询
#谁的工资大于able的工资
#方式一 自连接
SELECT e1.last_name,e1.salary
FROM employees e1 JOIN employees e2
ON e1.`salary`>e2.`salary`
AND e2.`last_name`='Abel';
#方式二 自连接
 SELECT last_name,salary
 FROM employees
 WHERE salary>(
		 SELECT salary
		FROM employees
		WHERE last_name='Abel'
		);

#找到job_id和141员工相同，salary 比143号员工多的员工姓名，job_id和工资
SELECT last_name,salary
FROM employees
WHERE job_id=(
		SELECT job_id#返回值
		FROM employees
		WHERE employee_id=141
		)
AND salary>(
		SELECT salary#返回值
		FROM employees
		WHERE employee_id=143);
		
#多行子查询
#查看公司那些员工的工资等于各个部门的最低工资
SELECT last_name,salary
FROM employees
WHERE salary IN (
		SELECT MIN(salary)
		FROM employees
		GROUP BY department_id
		);	
#返回其他job_id比job_id为'IT_PROG'部门任一员工低的的员工号
SELECT employee_id,last_name,salary
FROM employees
WHERE job_id <> 'IT_PROG'
AND salary < ANY (
		SELECT salary
		FROM employees
		WHERE job_id='IT_PROG'
		);
#返回其他job_id比job_id为'IT_PROG'部门全部员工低的的员工号
SELECT employee_id,last_name,salary
FROM employees
WHERE job_id <> 'IT_PROG'
AND salary < ALL (
		SELECT salary
		FROM employees
		WHERE job_id='IT_PROG'
		);
#返回平均工资最低的部门id
#方式一
SELECT department_id
FROM employees
GROUP BY department_id
HAVING AVG( salary)=(SELECT MIN(ave_salary)
			FROM(
				SELECT AVG(salary) ave_salary
				FROM employees
				GROUP BY department_id
				) ave_s)
#方式二				
SELECT department_id
FROM employees
GROUP BY department_id
HAVING AVG( salary)<= ALL(
			SELECT AVG(salary) ave_salary
			FROM employees
			GROUP BY department_id
				) 
				
				
				
#相关子查询
#查询员工本部门大于部门平均工资的员工
SELECT last_name,salary,department_id
FROM employees m
WHERE salary>(
	SELECT AVG(salary)
	FROM employees e2
	WHERE department_id=m.`department_id`
	);
#查询员工的id,salary,按照department_name排序
SELECT employee_id,salary
FROM employees m
ORDER BY(
	SELECT department_name
	FROM departments
	WHERE departments.`department_id`=m.`department_id`)
```

10.数据库增删改查

```mysql
#1.创建和管理数据库
#1.1创建数据库
#方式1：
CREATE DATABASE mytest1;
#方式2:显示指明了要创建的数据库的字符集
CREATE DATABASE mytest2 CHARACTER SET 'gbk';
#方式3：如果没存在就创建，如果存在就没创建
CREATE DATABASE IF NOT EXISTS mytest3;
#1.2管理当前数据库
#查看当前连接的数据库
SHOW DATABASES;

#切换数据库
USE atguigudb;

#查看当前数据库中保存的数据表
SHOW TABLES;

#查看当前使用的数据库
SELECT DATABASE() FROM DUAL;

#查看指定数据库底下保存的数据表
SHOW TABLE FROM mysql;

#1.3修改数据库
#更改数据库字符集
ALTER DATABASE mytest2 CHARACTER SET 'utf-8';

#1.4删除数据库
#1.方式1

 DROP DATABASE mytest1;
 
 #方式2
  DROP DATABASE IF NOT EXISTS  mytest1;
  
  #2.如何创建表
  USE atguigudb;
  CREATE TABLE IF NOT EXISTS myemp1(
  id INT,
  emp_name VARCHAR(15),
  hire_date DATE
  );
  #查看表的结构
   DESC myemp1;
   #查看创建表的语句结构
   SHOW CREATE TABLE myemp1;
   #方式2：基于现有的表
   CREATE TABLE IF NOT EXISTS myemp2
   AS
   SELECT employee_id,last_name,salary
   FROM employees;
   DESC myemp2;
   #创建表employee_blank,实现对employees表的复制
    CREATE TABLE IF NOT EXISTS employee_blank1
   AS
   SELECT *
   FROM employees
   WHERE 1=2;
   DESC employee_blank1;
   #3.修改表
   DESC myemp1;
   #3.1添加字段
   ALTER TABLE myemp1
   ADD salary DOUBLE(10,2);
   
   ALTER TABLE myemp1
   ADD phone_number VARCHAR(15) FIRST;#创建字段放在表的第一位
   
   ALTER TABLE myemp1
   ADD emil VARCHAR(40) AFTER salary;
   #3.2修改一个字段：数据类型，长度，默认值
   ALTER TABLE myemp1
   MODIFY emp_name VARCHAR(25)  DEFAULT 'aaaa';
  
   #3.3 重名名一个字段
   ALTER TABLE myemp1
   CHANGE salary monthly_salary DOUBLE(10,2);
   #3.4  删除一个字段
   ALTER TABLE myemp1
   DROP COLUMN emil;
   #重命名表
   #方式1
   RENAME TABLE myemp1
   TO myemp11;
    #方式2
    ALTER TABLE myemp11
    RENAME TO myemp12;
    #5.删除表
    DROP TABLE IF EXISTS myemp12;
    #6.清空表
    SELECT * FROM myemp2;
    TRUNCATE TABLE myemp2;
```

11.数据增删改

```mysql
#数据处理-增删改
#1.添加数据
USE atguigudb;
CREATE TABLE IF NOT EXISTS emp1(
id INT,
`name` VARCHAR(15),
hire_date DATE,
salary DOUBLE(10,2)
)
DESC emp1;

#方式一，一条一条添加
INSERT INTO emp1
VALUES (1,'Tom','2000-12-21',3400);
#指明字段赋值，没有指明的1字段为nul
INSERT INTO emp1(id,`name`,hire_date,salary);
VALUES (1,'Tom','2000-12-21',3400);
#同时插入多组数据
INSERT INTO emp1(id,salary,'name')
VALUES
(4,3400,'jerry'),
(5,5000,'jier');
#方式二，查询结果插入表中
INSERT INTO emp1(id,'name',salary,hire_date)
SELECT employee_id,last_name,salary,hire_date
FROM employees
WHERE department_id IN (60,70);

#更新数据
UPDATE emp1
SET hire_date=CURDATE()
where id=5;
#同时修改多个字段
UPDATE emp1
SET hie_date=CURDATE(),salary=6000
WHERE id%5=0;

#3.删除数据
DELETE FROM emp1
WHERE id=1;
#计算列
USE atguigudb;
CREATE TABLE test1(
a INT,
b INT,
c INT  GENERATED ALWAYS AS (a+b) VIRTUAL	
);
INSERT INTO test1(a,b)
VALUE (20,10);
SELECT *FROM test1;

UPDATE test1
SET a=100;

```

12.约束

```mysql
REATE DATABASE dbtest11;
USE dbtest11;
CREATE TABLE test1(
id INT,
NAME VARCHAR(10) NOT NULL,
email VARCHAR(25),
salary DECIMAL(10,2));

INSERT INTO test1
VALUE(1,'Tom','1453234',34000);
#错误例子
INSERT INTO test1(id,email)
VALUE(2,'14656');
#ALTER添加约束
ALTER TABLE test1
MODIFY email VARCHAR(25) NOT NULL;
#删除非空约束
ALTER TABLE test1
MODIFY email VARCHAR(25) NULL;

#唯一约束
 CREATE TABLE test2(
id INT UNIQUE,#列约束
last_name VARCHAR(15),
email VARCHAR(25),
salary DECIMAL(10,2),
CONSTRAINT uk_test2_email UNIQUE(email));
 #alter 添加约束
 ALTER TABLE test2
 ADD CONSTRAINT uk_test2_salary UNIQUE(salary);
 
 ALTER TABLE test2
 MODIFY CONSTRAINT uk_test2_lastname UNIQUE(last_name);
#复合唯一
 CREATE TABLE test3(
id INT ,
last_name VARCHAR(15),
email VARCHAR(25),
salary DECIMAL(10,2),
CONSTRAINT uk_test2_id_email UNIQUE(id,email));#两个都得不同

#删除唯一性约束
ALTER TABLE test3
DROP INDEX uk_test2_id_email;
DESC test3;


#主键约束=唯一约束+非空约束
CREATE TABLE test4(
id INT ,
last_name VARBINARY(15),
salary DECIMAL(10,2),
CONSTRAINT  pk_test4 PRIMARY KEY(id)
);#没必要起名字
DESC test4;
CREATE TABLE test5(
id INT ,
last_name VARBINARY(15),
salary DECIMAL(10,2),
CONSTRAINT  pk_test4 PRIMARY KEY(id,salary)
);#没必要起名字


#自增列,必须建立在主键约束上
CREATE TABLE test6(
id INT PRIMARY KEY AUTO_INCREMENT,
last_name VARCHAR(15));
#id,自己增加
INSERT INTO test6(id,last_name)
VALUES(-3,'tom'),
(-4,'jerry');
SELECT * FROM test6;


#外键约束
#先建立主表
CREATE TABLE dept1(
dept_id INT,
dept_name VARCHAR(15),
CONSTRAINT PRIMARY KEY(dept_id)
);
#建立从表
CREATE TABLE emp1(
emp_id INT PRIMARY KEY AUTO_INCREMENT,
emp_name VARCHAR(15),
department_id INT,
CONSTRAINT fk_emp1_dept_id FOREIGN KEY(department_id) REFERENCES dept1(dept_id)
);
DESC dept1;
DESC emp1;
#主表没有的，从表不能添加  
#约束等级
#on update cascade on delete set null 最好使用这个
CREATE TABLE dept2(
did INT PRIMARY KEY, #部门编号
dname VARCHAR(50)  #部门名称
);
CREATE TABLE emp2(
eid INT PRIMARY KEY,#员工编号
ename VARCHAR(5),#员工姓名
deptid INT,#员工所在部门
FOREIGN KEY (deptid) REFERENCES dept2(did) ON UPDATE CASCADE ON DELETE SET NULL
);
INSERT INTO dept2 
VALUES(1001,'教学部'),
(1002,'财务部'),
(1003,'咨询部');
INSERT INTO emp2
VALUES (1,'张三',1002),
(2,'李四',1001),
(3,'王五',1002);

UPDATE dept2
SET did=1004
WHERE did=1001;
SELECT  * FROM emp2;

#check约束
CREATE TABLE test10(
id INT,
last_name VARCHAR(15),
salary DECIMAL(10,2) CHECK(salary>2000)
);
INSERT INTO test10
VALUES (1,'tom',1000);#添加失败

#默认值约束
CREATE TABLE test11(
id INT,
last_name VARCHAR(15),
salary DECIMAL(10,2) DEFAULT 2000
);
```

13.视图

```mysql
#视图本质是存储的select语句，不存储数据，实际是操作基表数据
CREATE DATABASE dbtest14;
USE dbtest14;
CREATE TABLE emps
AS 
SELECT *
FROM atguigudb.`employees`;
CREATE TABLE depts
AS
SELECT *
FROM atguigudb.`departments`;

SHOW CREATE DATABASE dbtest14;
#创建视图
CREATE VIEW vu_emp1
AS
SELECT employee_id,last_name,salary
FROM emps;
 
 #如果存在就替换
CREATE OR REPLACE VIEW vu_emp2(emp_id,lname,sal)
AS
SELECT employee_id,last_name,salary
FROM emps;
#查看视图
#查看数据库的表对象和视图对象
SHOW TABLES;
#查看视图的结构
DESCRIBE vu_emp1;

#对视图进行增删改查
SELECT *FROM vu_emp1;

UPDATE vu_emp1
SET salary=20000
WHERE employee_id=101;

ALTER VIEW vu_emp1
AS 
SELECT employee_id
FROM emps;

DESC vu_emp1;
#删除视图
DROP VIEW vu_emp1;
```

14.存储过程和函数

```mysql
CREATE DATABASE dbtest15;
USE dbtest15;
CREATE TABLE employees
AS 
SELECT *
FROM atguigudb.`employees`;
CREATE TABLE departments
AS
SELECT *
FROM atguigudb.`departments`;
#创建存储过程
#实例1
DELIMITER $
CREATE PROCEDURE select_all_data()
BEGIN
	SELECT * FROM employees;
END $
DELIMITER;
#存储过程调用
CALL select_all_data();
#输出最小工资	
DELIMITER //
CREATE PROCEDURE show_min_salary (OUT ms DOUBLE)
BEGIN
	SELECT MIN(salary) INTO ms
	FROM employees;	

END //
DELIMITER;
CALL show_min_salary (@ms);	
SELECT @ms;

#查询某人工资
DELIMITER //
CREATE PROCEDURE show_people_salary (IN people VARCHAR(20))
BEGIN
	SELECT salary FROM employees
	WHERE last_name=people;	

END //
DELIMITER ;
CALL show_people_salary ('Abel');

DROP PROCEDURE  name_mag_em	
#查询员工领导的姓名
DELIMITER //
CREATE PROCEDURE name_mag_em (INOUT pname VARCHAR(20))
BEGIN
	SELECT last_name INTO pname
	FROM employees
	WHERE employee_id=(
	SELECT manager_id
	FROM employees
	WHERE last_name=pname
	);	

END //
DELIMITER ;
SET @pname='Abel';
CALL name_mag_em(@pname);
SELECT @pname;


#存储函数
DELIMITER //
CREATE FUNCTION fc_email_name (pname VARCHAR(20))
RETURNS VARCHAR(20)
	DETERMINISTIC
	CONTAINS SQL
	READS SQL DATA
	
BEGIN
	RETURN (SELECT email FROM employees
	WHERE last_name=pname	);

END //
DELIMITER ;
#调用
SELECT fc_email_name('Abel');

#存储函数和过程的查看和删除
SHOW CREATE PROCEDURE show_people_salary;
SHOW CREATE FUNCTION fc_email_name;
```

15.变量，循环和游标

```mysql
变量

#1.系统变量
#全局系统变量
SHOW GLOBAL VARIABLES;
#会话变量
SHOW SESSION VARIABLES;
#查看指定系统变量
SELECT @@global.max_connections;
SELECT @@session.pseudo_thread_id;
#修改系统变量 当服务器重启，又恢复到原始的值
SET @@global.max_connections=161;

#2.用户变量
#变量定义
SET @m1=1;
SET @m2=2;
SET @sum=@m1+@m2;
#准备工作
CREATE DATABASE dbtest16;
USE dbtest16;
CREATE TABLE employees
AS 
SELECT *
FROM atguigudb.`employees`;
CREATE TABLE departments
AS
SELECT *
FROM atguigudb.`departments`;
#测试
SELECT @count:=COUNT(*) FROM employees;
SELECT AVG(salary) INTO @avg_sal FROM employees;
SELECT @avg_sal;
#局部变量
DELIMITER //
CREATE PROCEDURE test_var()
BEGIN
	DECLARE a INT DEFAULT 0;
	DECLARE b INT;
	DECLARE emp_name VARCHAR(25);

	SET a=1;
	SET b:=2;
	SELECT last_name INTO emp_name FROM employees WHERE employee_id=101;
	
	
	SELECT a,b,emp_name;
END //
DELIMITER;
	CALL test_var();
DROP PROCEDURE test_var;
CALL test_var;
#流程控制
#if结构
DELIMITER //
CREATE PROCEDURE test_if()
BEGIN
	DECLARE stu_name VARCHAR(25) DEFAULT 'acs';
	DECLARE age INT  DEFAULT 40;
	IF age >40
		THEN SELECT '老年' INTO stu_name;
	ELSEIF age<40
		THEN SELECT '壮年' INTO stu_name;
	ELSE
		SELECT '少年' INTO stu_name;
	END IF;
	SELECT stu_name;
END //
DELIMITER ;
CALL test_if;
DROP PROCEDURE test_if;

#case语句

DELIMITER //
CREATE PROCEDURE test_case()
BEGIN
	DECLARE var INT DEFAULT 2;
	CASE var
		WHEN 1 THEN SELECT 'var=1';
		WHEN 2 THEN SELECT 'var=2';
		WHEN 3 THEN SELECT 'var=3';
		ELSE SELECT 'other value';
	
	END CASE;
	
END //
DELIMITER ;
CALL test_case;
#循环结构

#loop
DELIMITER //
CREATE PROCEDURE test_loop()
BEGIN
	DECLARE num INT DEFAULT 1;
	loop_label:LOOP
		SET num=num+1;
		IF num>=10
			THEN LEAVE loop_label;
		END IF;
	END LOOP loop_label;
	
	SELECT num;
	
END //
DELIMITER ;
CALL test_loop;

#while
DELIMITER //
CREATE PROCEDURE test_while()
BEGIN
	DECLARE num INT DEFAULT 1;
	WHILE num<=10  DO
		SET num=num+1;
	END WHILE ;
	
	SELECT num;
	
END //
DELIMITER ;
CALL test_while;

#repeat
DELIMITER //
CREATE PROCEDURE test_repeat()
BEGIN
	DECLARE num INT DEFAULT 1;
	REPEAT
		SET num=num+1;
		UNTIL num>10
	END REPEAT;
	SELECT num;
	
END //
DELIMITER ;
CALL test_repeat;
 
#leave使用  使用leave 得加标签名,和break作用差不多，
DELIMITER //
CREATE PROCEDURE leave_begin(IN num INT)
begin_label:BEGIN
	IF num<=0
		THEN LEAVE begin_label;
	ELSE 
			SELECT '结束';
	END IF;
	
END //
DELIMITER ;
CALL leave_begin(1);

#iterate 只能在循环里面用 就是continue
DELIMITER //
CREATE PROCEDURE test_iterate()
BEGIN
	DECLARE num INT DEFAULT 1;
	while_lable:WHILE num<=10  DO
		IF num=50
			THEN ITERATE while_lable; 
		END IF;
		SET num=num+1;
	END WHILE ;
	
	SELECT num;
	
END //
DELIMITER ;
 
 
 #游标
 DELIMITER //
 CREATE PROCEDURE get_sum(IN li_sa DOUBLE,OUT to_count INT)
 BEGIN
	DECLARE sum_sal DOUBLE DEFAULT 0.0;
	DECLARE emp_sal DOUBLE DEFAULT 0.0;
	DECLARE emp_count INT DEFAULT 0;
	DECLARE emp_cursor CURSOR FOR SELECT salary FROM employees
		ORDER BY salary DESC;
	OPEN emp_cursor;
	REPEAT
		FETCH emp_cursor INTO emp_sal;
		SET sum_sal=sum_sal+emp_sal;
		SET emp_count=emp_count+1;
		UNTIL sum_sal>=li_sa
	END REPEAT;
	SET to_count=emp_count;
	CLOSE emp_cursor;
 END //
 DELIMITER ;
 DESC employees;
 SET @lim=100000;
 SET @to_count=0;
 CALL get_sum(@lim,@to_count)
 SELECT @to_count
```

16.触发器

```mysql
#触发器

CREATE DATABASE dbtest17;
USE dbtest17;
CREATE TABLE test_trigger(
id INT PRIMARY KEY AUTO_INCREMENT,
t_note VARCHAR(30)
);
CREATE TABLE test_rigger_log(
id INT PRIMARY KEY AUTO_INCREMENT,
t_log VARCHAR(30)
);
#创建触发器
DELIMITER //
CREATE TRIGGER before_insert_tri
BEFORE INSERT ON test_trigger
FOR EACH ROW
BEGIN
	INSERT INTO test_rigger_log(t_log)
	VALUES(';;;;');
END //
DELIMITER ;
DROP TRIGGER before_insert_tri;
INSERT INTO test_trigger(t_note)
VALUES('30');
SELECT * FROM test_trigger;
SELECT * FROM test_rigger_log;
#课后练习
#创建表格emps
CREATE TABLE emps
AS 
SELECT employee_id,last_name,salary
FROM atguigudb.`employees`;
#复制emps的空表emps——back,只有表的结构
CREATE TABLE emps_back
AS
SELECT * FROM emps
WHERE 1=2;
#创建触发器
DELIMITER //
CREATE TRIGGER emps_inser
AFTER INSERT ON emps
FOR EACH ROW
BEGIN
	INSERT INTO emps_back(employee_id,last_name,salary)
	VALUES(NEW.employee_id,NEW.last_name,NEW.salary);#NEW就是新插入的数据
END //
DELIMITER ;

INSERT INTO emps
VALUES(301,'tom',3600)


SELECT * FROM emps_back

```

