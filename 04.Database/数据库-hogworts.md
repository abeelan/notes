## 简介

**数据（Data）**是描述事物的符号记录，是指利用物理符号记录下来的、可以鉴别的信息。

**数据库（Database, DB）**是指长期储存在计算机中的有组织、可共享的数据集合。

**数据库管理系统（DBMS）**是专门用于建立和管理数据库的一套软件，介于应用程序和操作系统之间。



## 数据库分类

- 关系型数据库（RDB - Relationship Database）
    - Mysql、Oracle、Postgres、SQLLite、SQLServer
    - 使用场景：数据处理较复杂、数据量不是特别大、对安全性要求比较高、数据格式单一
- 非关系型数据库（NoSQL - Not only sql）
    - MongoDB、Redis、HBase、Neo4j（图分析数据库）
    - 数据模型比较简单、需要灵活性更强的 IT 系统、对数据库性能要求较高、不需要高度的数据一致性



## MySQL 安装与配置

```bash
$ mysql -V

# 开启/关闭 mysql 服务
$ net start mysql
$ net stop mysql

# 登录
$ mysql -h {主机IP} -u {用户名} -p {密码}

# 修改密码
$ alter user 'root'@'localhost' identified by '新密码';

# 退出 
$ exit 
```



## 数据库客户端工具

- workBench；下载地址：https://dev.mysql.com/downloads/workbench/

- Navicat（付费）



## 表

> 包含数据库中所有数据的数据库对象

表名：每个表的唯一标识

模式：schema，关于数据库和表的布局及特性的信息

列：表中每列称为一个字段

行：表中的一条记录



## SQL

##### SQL 是什么？

- 结构化查询语言，简称 SQL
- 一种特殊目的的编程语言
- 一种数据库查询和程序设计语言
- 用于存取数据以及查询、更新和管理关系数据库系统

##### 通用语法

- 可以单行或者多行书写，以分号结尾
- 可以使用空格和缩进来增加语句的可读性
- 不区分大小写，一般关键字大些，数据库名、表名、列名小写
- 注释方式：`--`、`#`（仅 MySQL） 单行注释；`/* */` 多行注释

##### 分类

- 数据定义语言（DDL）：用来定义数据库对象，比如数据库、表、列等
- 数据操作语言（DML）：用来对数据库中表的记录进行更新
- 数据查询语言（DQL）：用来查询数据库中表的记录
- 数据控制语言（DCL）：用来定义数据库的访问权限和安全级别及创建用户



### DDL 数据库操作

#### 创建

```mysql
# 创建数据库语法
# {} 必选项；｜ 或；[] 可选项
CREATE {DATABASE|SCHEMA} [IF NOT EXISTS] 数据库名 CHARACTER SET [=] 字符集; 
```



创建数据库注意事项：

- 不能与其他数据库重名
- 名称可以由任意字母、阿拉伯数字、下划线和 `$` 组成，但不能使用单独的数字
- 名称最长可为 `64` 个字符，别名最长为 `256` 个字符
- 不能使用 MySQL 关键字作为数据库名
- 建议采用小写来定义数据库名



```mysql
-- 创建数据库
CREATE DATABASE test_db;
-- 指定字符集为 utf-8
CREATE DATABASE test_db2 CHARACTER SET utf8;
-- 创建数据库前判断是否存在同名数据库
CREATE DATABASE IF NOT EXISTS test_db3 CHARACTER SET utf8;
```



#### 查看

```mysql
-- 查看所有数据库
SHOW DATABASES;
-- 指定为当前数据库
USE 数据库名;
-- 查看数据库定义信息
SHOW CREATE DATABASE 数据库名;
```



#### 修改

```mysql
-- 修改数据库相关参数，如果不指定数据库名则表示修改当前的数据库
ALTER {DATABASE} [数据库名] CHARACTER SET [=] 字符集;

-- 实例
CREATE DATABASE db1 CHARACTER SET gbk;
ALTER DATABASE db1 CHARACTER SET utf8;
```



#### 删除

```mysql
-- 删除数据库; 删除后数据结构和数据都没了
DROP DATABASE [IF EXISTS] 数据库名;
```



### DDL 表操作

MySQL 的数据类型

- 数字类型（大概下面三种，长短整型、双精度浮点型等不详细列了）
    - INT 整型
    - FLOAT 浮点型
    - BOOLEAN 布尔值
- 字符串类型
    - CHAR 固定长度（1-255）
    - VARCHAR 长度可变，不超过 255 个字符
    - TEXT 最大长度为 64K 的变长文本
- 日期和时间类型
    - DATE 日期
    - TIME 时间
    - DATETIME 日期和时间
    - TIMESTAMP 时间戳
    - YEAR 年份

#### 创建

```mysql
-- 创建表；列名在表中唯一
CREATE TABLE 数据表名 (
	列名 数据类型 [NOT NULL | NULL] [DEFAULT 默认值] [AUTO_INCREMENT] [PRIMARY KEY] [注释],
    ...
);

-- AUTO_INCREMENT 是否自动编号
-- PRIMARY KEY 是否为主键

-- 复制表(结构)
CREATE TABLE 数据表名 {LIKE 源数据表名 | (LIKE 源数据表名)};
```

实例

```mysql
-- 切换到数据库
USE db1;

-- 创建学员表
CREATE TABLE student (
	id INT,
    name VARCHAR(20)
);

-- 复制学员表
CREATE TABLE student_cpy LIKE student;
```



#### 查看

```mysql
-- 查看表名
SHOW TABLES;

-- 查看表结构
DESCRIBE 数据表名;
-- 简写
DESC 数据表名;
DESC 数据表名 列名;
```



#### 修改

```mysql
-- 添加新列
ALTER TABLE 表名 ADD 列名 列属性;

-- 修改列定义
ALTER TABLE 表名 MODIFY 列名 列属性;

-- 修改列名
ALTER TABLE 表名 CHANGE 旧列名 新列名 类型;
```

实例

```mysql
-- 切换数据库
USE db1;
-- 添加列
ALTER TABLE student ADD email VARCHAR(50) NOT NULL;
-- 修改列定义
ALTER TABLE student ADD score VARCHAR(10) NOT NULL;
ALTER TABLE student MODIFY score INT;
-- 修改列名
ALTER TABLE student CHANGE COLUMN name stu_name VARCHAR(30) DEFAULT NULL;
-- 查看表结构，验证修改生效
DESC student;
```



#### 删除列

```mysql
ALTER TABLE 表名 DROP 列名;

ALTER TABLE student DROP score;
```



#### 修改表名

```mysql
-- 方式一
ALTER TABLE 旧表名 RENAME AS 新表名;

-- 方式二
RENAME TABLE 旧表名 TO 新表名;
```



#### 删除表

```mysql
DROP TABLE [IF EXISTS] 数据表名;
```



### DML 表数据操作

#### 插入

```mysql
-- 插入数据 列名和值一一对应
INSERT INTO 数据表名 (列名1, 列名2, ...) VALUES (值1, 值2, ...);
```



注意事项：

- 值与字段必须对应，个数相同且数据类型相同
- 值的数据大小，必须在字段指定的长度范围内
- VARCHAR、CHAR、DATE 类型的值，必须使用单引号括起来
- 如果要插入空值，可以忽略不写，或者插入 NULL
- 如果插入指定字段的值，必须写上列名



#### 修改

```mysql
UPDATE 数据表名 SET 列名1=值1 [, 列名2=值2, ...] [WHERE 条件表达式];
```



#### 删除

```mysql
-- 通过 delete 语句; 未指定条件则删除表中全部数据
DELETE FROM 表名 [WHERE 条件表达式];

-- 通过 truncate table 语句删除表中全部数据
TRUNCATE TABLE 表名;
```

如果需要删除整张表，推荐使用 `TRUNCATE` 语句：

- DELETE：每条数据都会执行一次删除，效率低
- TRUNCATE：删除整张表，然后重新创建空表，效率高



**实例**

```mysql
-- 切换数据库
USE db1;

-- 创建 user 表
CREATE TABLE user(
	id INT,
    name VARCHAR(20),
    age INT,
    sex CHAR(1),
    address VARCHAR(40)
);

/* 插入 */
-- 插入一条完整的数据
INSERT INTO user (id, name, age, sex, address) VALUES (1, '张三', 20, '男', '北京');

-- 插入一条完整的数据，可以不用写列名，值按照表的列的顺序插入
INSERT INTO user VALUES (2, '李四', 19, '女', '上海');

-- 插入表中某几列的值，没有值的列自动为 NULL
INSERT INTO user (id, name, address) VALUES (3, '王五', '深圳');

-- 插入多条数据
INSERT INTO user (id, name, address) VALUES 
	(4, '赵六', '天津'),
	(5, '周七', '杭州');
	
/* 修改 */
-- 不带条件修改，将所有的性别改为女
UPDATE user SET sex='女';

-- 带条件修改，id=3 的用户：性别修改为男
UPDATE user SET sex='男' WHERE id=3;

-- 修改多个列的数据，id=2 的用户：年龄修改为 30，地址修改为 北京
UPDATE user SET age=30, adress='北京' WHERE id=2;

/* 删除 */
-- 删除 id=1 的用户数据
DELETE FROM user WHERE id=1;

-- 删除表中所有的数据
DELETE FROM user;
TRUNCATE TABLE user;
```



### DQL 表查询操作

在进行查询前，先进行数据准备工作，使用开源的数据示例。

> https://github.com/datacharmer/test_db
>
> 该数据库包含大约 300,000 条员工记录和 280 万条工资条目。导出数据为 167 MB，对于测试来说足够。 

```bash
# 这里通过 docker 启动 mysql:5.7 的容器进行测试

# 1、下载数据文件
$ cd workspace/github
$ git clone git@github.com:datacharmer/test_db.git

# 2、启动 mysql 容器，挂载数据文件到容器内
$ docker pull mysql:5.7
$ docker run --name=mysql -it -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e MYSQL_ROOT_HOST=% -v /workspace/github/test_db:/opt -d mysql:5.7

# 3、导入数据库
$ docker exec -it mysql /bin/bash
$ mysql -h 127.0.0.1 -u root -p < /opt/employees.sql
Enter password: root

# 等待导入完成，大概 1 分钟+
```



#### 单表查询

> 从一张表中查询所需要的数据。

```mysql
-- 查询全部数据
SELECT * FROM 表名;

-- 实例：查询部门表中的信息
SELECT * FROM departments;
```

```mysql
-- 查询多个字段（列）, 多个列中间用逗号分隔
SELECT 列名 FROM 表名;

-- 实例：查询部门名称
SELECT dept_name FROM departments;
```

```mysql
/* 起别名 */
-- 为表起别名
SELECT 列名 FROM 表名 表别名;
-- 为字段起别名
SELECT 列名 AS 别名 FROM 表名;

-- 实例：查询员工信息，并将列名改为中文，表名改为 emp
SELECT
	emp_no AS '编号',
	first_name AS '名',
	last_name AS '姓',
	gender AS '性别',
	hire_date AS '入职时间' 
FROM
	employees emp;
```

```mysql
-- 去重
SELECT DISTINCT 列名 FROM 表名;

-- 实例：去掉重复职级信息
SELECT DISTINCT title FROM titles;
```

```mysql
-- 查询结果参与运算
SELECT (列名 运算表达式) FROM 表名;

-- 实例：所有员工的工资 +1000 进行显示
SELECT emp_no, salary + 1000 FROM salaries;
```



#### 条件查询

```mysql
-- 条件查询
SELECT 列名 FROM 表名 WHERE 条件表达式;
```



##### 比较运算法

- 大于（>）、小于（<）、>=、<=
- 等于（=）如果是字符，则比较 ASCII 码
- 不等于（<>、!=）
- 范围限定（BETWEEN...AND...）
- 子集限定（IN）
- 模糊查询（LIKE '%or%'）
- 为空（IS NULL）

```mysql
-- 实例：比较大小，查询出生日期晚于 1965-01-01 的员工编号、姓名和生日
SELECT emp_no, first_name, birth_date FROM employees  WHERE birth_date > '1965-01-01';
```

```mysql
-- 使用 BETWEEN 进行模糊查询
WHERE 列名 [NOT] BETWEEN 起始表达式 AND 结束表达式

-- 实例：查询年薪介于 70000 到 70003 之间的员工编号和年薪
SELECT emp_no, salary FROM salaries WHERE salary BETWEEN 70000 AND 70003;
```

```mysql
-- 使用 IN 进行模糊查询, 常量列表使用逗号隔开
WHERE 列名 IN (常量列表)

-- 实例：查询入职日期为 1995-01-27 和 1995-03-20 的员工信息
SELECT * FROM employees WHERE hire_date IN ('1995-01-27', '1995-03-20');
```

```mysql
-- 判断是否为空
WHERE <列名> IS [NOT] NULL

-- 实例：查询性别为空的员工信息
SELECT * FROM employees WHERE gender IS NULL;  -- 没有为空的数据
SELECT * FROM employees WHERE gender IS NOT NULL;  -- 全部员工数据
```



##### 逻辑运算法

- AND 或 &&：多个条件同时成立
- OR 或 ||：多个条件任一成立
- NOT：不成立

```mysql
-- 查询名字为 Lillian 并且姓氏为 Haddadi 的员工信息
SELECT * FROM employees WHERE first_name = 'Lillian' AND last_name = 'Haddadi';

-- 查询名字为 Lillian 或者姓氏为 Terkki 的员工信息
SELECT * FROM employees WHERE first_name = 'Lillian' OR last_name = 'Terkki';

-- 查询名字为 Lillian 并且性别不是女的员工信息
SELECT * FROM employees WHERE first_name = 'Lillian' AND NOT gender = 'F';
```



##### 通配符

- %：匹配任意多个字符
- _：匹配一个字符

```mysql
-- 查询名字中包含 fai 的员工信息
SELECT * FROM employees WHERE first_name LIKE '%fai%';

-- 查询名字中 fa 开头的且长度为 3 位的员工信息
SELECT * FROM employees WHERE first_name LIKE 'fa_';
```



#### 排序

排序语法

- ASC：升序（默认）
- DESC：降序

```mysql
-- 对查询结果进行排序
SELECT 列名 FROM 表名 [WHERE 条件表达式] 
	ORDER BY 
	列名1 [ASC | DESC], 列名2 [ASC | DESC];
```



##### 单列排序

```mysql
-- 使用 salary 字段，对 salaraies 表数据进行 升序/降序 排序
SELECT * FROM salaries ORDER BY salary;
SELECT * FROM salaries ORDER BY salary DESC;

-- 查询员工的编号和入职日期，按照员工入职日期从晚到早排序
SELECT emp_no, hire_date FROM employees ORDER BY hire_date DESC;
```



##### 组合排序

同时对多个字段进行排序，如果第一个字段相同，就按照第二个字段进行排序。

```mysql
-- 在入职时间排序的基础上，再使用 emp_no 进行降序排序
SELECT emp_no, hire_date FROM employees ORDER BY hire_date DESC, emp_no DESC;
```



#### 聚合函数

- COUNT()：统计指定列不为 NULL 的记录行数
- MAX()：计算指定列的最大值
- MIN()：计算指定列的最小值
- SUM()：计算指定列的数值和
- AVG()：计算指定列的平均值

```mysql
-- 聚合查询语法
SELECT 聚合函数(列名) FROM 表名;

-- 实例
-- 查询职级名称为 Senior Engineer 的员工数量
SELECT COUNT(title) FROM titles WHERE title = 'Senior Engineer';
SELECT COUNT(*) FROM titles WHERE title = 'Senior Engineer';

-- 查询员工编号为 10002 的员工的最高年薪
SELECT MAX(salary) FROM salaries WHERE emp_no = 10002;

-- 查询最低年薪
SELECT MIN(salary) FROM salaries WHERE emp_no = 10002;

-- 查询薪资总和
SELECT SUM(salary) FROM salaries WHERE emp_no = 10002;

-- 计算平均薪资
SELECT AVG(salary) FROM salaries WHERE emp_no = 10002;
```



#### 分组

分组查询语法

- 分组列：按哪些列进行分组
- HAVING：对分组结果再次过滤

```mysql
-- 分组查询语法
SELECT (分组列 ｜ 聚合函数) FROM 表名 GROUP BY 分组列 [HAVING 条件];
```

实例

```mysql
-- 查询每个员工的薪资和
SELECT emp_no, SUM(salary) FROM salaries GROUP BY emp_no;

-- 查询员工编号小于 10010 且薪资和小于 400000 的员工的薪资和
SELECT emp_no, SUM(salary) FROM salaries WHERE emp_no < 10010 GROUP BY emp_no HAVING SUM(salary) < 400000;
```

子句区别

- WHERE 子句：从数据源中去掉不符合搜索条件的数据
- GROUP BY 子句：搜集数据行到各个组中，统计函数为各个组计算统计值
- HAVING 子句：去掉不符合其组搜索条件的各行数据行



#### LIMIT 关键字

限制查询结果的数量

```mysql
-- 限制查询结果行数
SELECT 列名1, 列名2 FROM 表名 LIMIT [开始的行数], <查询记录的条数>;

-- 使用 OFFSET 关键自指定开始的行数
SELECT 列名1, 列名2 FROM 表名 LIMIT <查询记录的条数> OFFSET <开始的行数>;

-- 开始的行数：从 0 开始，默认为 0
-- 查询记录的条数：返回的行数
```

实例

```mysql
-- 展示前 10 条员工信息
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 0, 10;
SELECT * FROM employees LIMIT 10 OFFSET 0;

-- 显示年薪从高到低排序，第 15 位到第 20 员工的编号和年薪
SELECT emp_no, salary FROM salaries ORDER BY salary DESC LIMIT 14, 6;
SELECT emp_no, salary FROM salaries ORDER BY salary DESC LIMIT 6 OFFSET 14;
```



#### 单表查询总结

```mysql
SELECT DISTINCT 列名 
FROM 表名
WHERE 查询条件表达式
GROUP BY 分组的列名
HAVING 分组后的查询条件表达式
ORDER BY 排序的列名 [ASC | DESC]
LIMIT 开始的行数, 查询记录的条数;
```

![SQL 语句执行顺序](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220507113411670.png)

## SQL 约束

对表中的数据进行进一步的限制，保证数据的正确性、有效性、完整性。违反约束的不正确数据无法插入到表中。常见的约束：

- 主键：PRIMARY KEY
- 非空：NOT NULL
- 唯一：UNIQUE
- 默认：DEFAULT
- 外键：FOREIGN KEY



### 主键约束

主键：一列（或一组列），其值能够唯一标识表中的每一行

特点：不可重复，唯一，非空

语法：列名 字段类型 PRIMARY KEY



##### 创建带有主键的表

```mysql
-- 创建一个带有主键的表
CREATE TABLE emp1(
    -- 设为主键后自动非空
	eid INT PRIMARY KEY,
	ename VARCHAR(20),
	gender CHAR(1)
);

-- 向已有的表中添加主键
CREATE TABLE emp2(
	eid INT,
	ename VARCHAR(20),
	gender CHAR(1)
);
ALTER TABLE emp2 ADD PRIMARY KEY(eid);
-- 向 emp1 中添加数据，验证主键非空和不能重复的特性
INSERT INTO emp1 VALUES(1, 'zhangsan', 'M');
-- 插入空值插入报错，主键不能为空
-- INSERT INTO emp1 VALUES(NULL, 'lisi', 'W');
-- 插入重复数据报错，主键不能重复
-- INSERT INTO emp1 VALUES(1, 'lisi', 'F');
```



##### 主键自增

由于主键的特性，每次人为按顺序添加难免出错，可以使用自增的属性。

```mysql
-- 创建主键自增
-- AUTO_INCREMENT: 表示自动增长（必须整数类型的字段），默认从 1 开始
CREATE TABLE emp3(
	eid INT PRIMARY KEY AUTO_INCREMENT,
	ename VARCHAR(20),
	gender CHAR(1)
);
INSERT INTO emp3 VALUEs(1, 'zhangsan', 'M');
-- 只需要添加名字和性别即可，id 会自增
INSERT INTO emp3(ename, gender) VALUES('lisi', 'M'), ('wangwu', 'F');

-- 修改主键自增起始值为 100
CREATE TABLE emp4(
	eid INT PRIMARY KEY AUTO_INCREMENT,
	ename VARCHAR(20),
	gender CHAR(1)
)AUTO_INCREMENT=100;
# DELETE FROM emp4;    删除表后，添加数据会继续之前的编号往上自增
# TRUNCATE TABLE emp4; 先删除整表后重新创建，自增会从起始值开始
```



##### 删除主键约束

```mysql
-- 删除表中的主键
ALTER TABLE 表名 DROP PRIMARY KEY;

-- 实例：删除表2 的主键
ALTER TABLE emp2 DROP PRIMARY KEY;
```



##### 选择主键原则

- 针对业务设计主键，建议每张表都设计一个主键
- 主键可以没有业务意义，只需保证不重复



### 非空约束

特点：某一列不允许为空

语法：列名 字段类型 NOT NULL

```mysql
-- 创建非空约束的表
CREATE TABLE emp5 (
	eid INT PRIMARY KEY AUTO_INCREMENT,
	ename VARCHAR(20) NOT NULL,
	gender CHAR(1)
);
INSERT INTO emp5(ename, gender) VALUES('zhangsan', 'F');
-- 报错：Field 'ename' doesn't have a default value, Time: 0.001000s
INSERT INTO emp5(gender) VALUES('F');
```



### 唯一约束

- 表中的某一列的值不能重复
- 对 NULL 不做唯一的判断
- 语法：列名 字段类型 UNIQUE

```mysql
-- 创建带有唯一约束的表
CREATE TABLE emp6 (
	eid INT PRIMARY KEY AUTO_INCREMENT,
	ename VARCHAR(20) UNIQUE,
	gender CHAR(1)
);
INSERT INTO emp6(ename, gender) VALUES('zhangsan', 'F');
-- 报错：Duplicate entry 'zhangsan' for key 'ename', Time: 0.004000s
INSERT INTO emp6(ename, gender) VALUES('zhangsan', 'F');
```



主键约束与唯一约束的区别：

- 主键约束：唯一且不能为空
- 唯一约束：唯一但可以为空
- 一个表中只能有一个主键，但是可以有多个唯一约束



### 默认值约束

默认值约束：用来指定某列的默认值

语法：列名 字段类型 DEFAULT 默认值

```mysql
-- 创建带有默认值的表
CREATE TABLE emp7 (
	eid INT PRIMARY KEY AUTO_INCREMENT,
	ename VARCHAR(20),
	gender CHAR(1) DEFAULT 'F'
);
-- 使用默认值，可以通过 DEFAULT 指定，也可以不写
INSERT INTO emp7(ename, gender) VALUES('zhangsan', DEFAULT);
INSERT INTO emp7(ename) VALUES('lisi');
-- 不使用默认值
INSERT INTO emp7(ename, gender) VALUES('wangwu', 'M');
```



### 外键约束

课程名称： SQL 约束 - 外键约束

TODO  不理解，等查阅完基础资料再来。

##### 创建表的同时创建外键约束

```mysql
-- 创建主表
CREATE TABLE tester (
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20),
	team_id INT,
    -- 添加外键及外键约束
	CONSTRAINT tester_team FOREIGN KEY(team_id) REFERENCES tester(id)
);
```



修改外键约束



删除外键约束



级联删除

```mysql
```



## 多表

> 在数据库设计中使用多张表格来实现数据存储的要求，在实际的项目中，数据量大且复杂，需要分库分表。

分表：按照一定的规则，对原有的数据库和表进行拆分，表与表之间可以通过外键建立连接。



实例：创建一张员工信息表，包含字段：

- eid：员工 ID（自增主键）
- ename：员工姓名
- age：年龄
- gender：性别
- dept_name：部门名称
- dept_id：部门 ID
- dept_manager：部门主管
- dept_location：所在地点



```mysql
-- 创建单表
CREATE TABLE emp(
	eid INT PRIMARY KEY AUTO_INCREMENT,
	ename VARCHAR(20),
	age INT,
	gender VARCHAR(5),
	dept_name VARCHAR(10),
	dept_id INT,
	dept_manager VARCHAR(10),
	dept_location VARCHAR(10)
);
INSERT INTO emp VALUES(1, 'zhangsan', 20, '', 'qa', 1, 'qa_leader', 'beijing');
```



当插入数据时，所有的数据都需要添加部门名称、部门经理等重复性信息，所以需要使用多表设计模式。

- 将数据拆分为员工信息表`table_emp`和部门信息表`table_dept`
- 两个表之间通过部门 ID`dept_id`字段连接

```mysql
CREATE TABLE table_emp(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20),
	age INT,
	gender VARCHAR(5),
	dept_id INT
);

CREATE TABLE table_dept(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(10),
	manager VARCHAR(10),
	location VARCHAR(10)	
);

INSERT INTO table_emp VALUES(1, 'zhangsan', 18, 'M', 1);
INSERT INTO table_emp VALUES(2, 'lisi', 19, 'F', 2);
INSERT INTO table_dept VALUES(1, 'qa', 'leader', 'beijing');
INSERT INTO table_dept VALUES(2, 'rd', 'rd_leader', 'beijing');
```



多表的优点：

- 简化数据
- 提高复用性
- 方便权限控制
- 提高系统的稳定性和负载能力



### 多表关系简介

##### 一对多

定义：主表的一条记录可以对应从表的多条记录。在一对多关系中，多的表定为从表，设置外键指向主表。



比如：部门表、员工表，一个部门可以对应多个员工。建表时，员工表就应作为从表，设置外键指向部门表的主键。



##### 多对多

定义：主表的多条记录可以对应从表的多条记录。需要创建第三张表作为中间表，中间表需要包含两种表的主键。



比如：商品信息表、客户表、订单表。



##### 一对一

定义：从表的一条记录对应主表的一条记录，这种对应关系的数据，通常放在单表里。



比如：员工信息表与身份证、联系方式。



### 多表查询

##### 笛卡尔积查询

通过查询多张表格获取数据，至少涉及两张表

```mysql
-- 创建部门表，插入三条数据
CREATE TABLE dept(
	id INT PRIMARY KEY AUTO_INCREMENT,
	dept_name VARCHAR(20),
	dept_manager VARCHAR(20),
	dept_location VARCHAR(20)
);

INSERT INTO dept VALUES (1, '研发部', '研发小张', '北京');
INSERT INTO dept VALUES (2, '运营部', '运营小李', '上海');
INSERT INTO dept VALUES (3, '销售部', '销售小王', '深圳');

-- 创建员工信息表，添加外键约束，允许级联删除，并对三个部门插入对应的员工信息
CREATE TABLE worker(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20),
	gender VARCHAR(10),
	dept_id INT,
	salary INT,
	-- 添加外键约束，级联删除
	CONSTRAINT FK FOREIGN KEY(dept_id) REFERENCES dept(id) ON DELETE CASCADE
);

INSERT INTO worker VALUES (1, '虞姬', '女', 2, 10000);
INSERT INTO worker VALUES (2, '项羽', '男', 1, 18000);
INSERT INTO worker VALUES (3, '后羿', '男', 3, 28000);


-- 查询表
SELECT * FROM dept;
SELECT * FROM worker;
-- 笛卡尔积查询，会出现很多无效数据
SELECT * FROM dept, worker;
-- 正确查询
SELECT * FROM dept, worker WHERE dept.id=worker.dept_id;

-- 查询出运营部的部门信息及该部门下的员工信息
SELECT * FROM dept, worker WHERE dept.id=worker.dept_id AND dept.id=2;
```



##### 内连接查询

内连接（INNER JOIN）：使用比较运算符进行表间某（些）列数据的比较操作，并列出这些表中与连接条件相匹配的数据行，组成新的记录。匹配上显示，匹配不上不显示。按语法结构分为：隐式内连接和显式内连接。

- 隐式内连接：使用 where 条件过滤无用数据，没有出现 inner join 关键字

- 显式内连接：添加 inner join 关键字

```mysql
-- 隐式内连接
SELECT [字段名称] FROM [表名] WHERE [条件];
-- 显示内连接
SELECT [字段名称] FROM [主表名] INNER JOIN [从表名] ON [条件];

SELECT worker.id, name, dept_location FROM worker, dept WHERE worker.dept_id=dept.id AND dept_name="运营部";

SELECT worker.id, name, dept_location FROM dept INNER JOIN worker ON dept.id=worker.dept_id AND dept_name="运营部";
```



##### 外连接查询

外连接：查询多个表中相关联的行，有时候需要包含没有关联的行中数据，即返回查询结果集合中不仅包含符合连接条件的行，还包括左表（左连接）、右表（右连接）中的所有数据行。

- 左外连接（LEFT OUTER JOIN），OUTER 可以省略，以左表为基准匹配右表的数据，确保左表数据都展示，右表中没有的项，显示为空。
- 右外连接（RIGHT OUTER JOIN），OUTER 可以省略，以右表为基准匹配左表的数据，确保右表数据都展示，左表中没有的项，显示为空。

```mysql
-- 左连接
SELECT [字段名称] FROM [左表] LEFT JOIN [右表] ON [条件];

-- 右连接
SELECT [字段名称] FROM [左表] RIGHT JOIN [右表] ON [条件];
```



##### 子查询

子查询指一个查询语句嵌套在另一个查询语句内部，在 SELECT 子句中先计算子查询，子查询的结果作为外层另一个查询的过滤条件，查询可以基于一个表或者多个表。



子查询作为过滤条件时需要用**小括号**包裹。



常见的三类子查询如下：



from 型

将子查询的结果作为父查询的表来使用。子查询是一张多行多列的表，将子查询作为父查询的表来嵌套查询，子查询必须用小括号包裹且有别名。

```mysql
-- 计算出各部门男性员工的人数
-- 最终查询字段部门名称、男性员工人数
-- 子查询：查询公司所有性别为男的员工信息
SELECT * FROM worker WHERE gender="男";

SELECT 
	dept_name AS '部门名称', COUNT(dept_id) AS '部门人数' 
FROM 
	(SELECT * FROM worker WHERE gender="男") AS male_worker INNER JOIN dept 
ON 
	male_worker.dept_id=dept.id 
GROUP BY 
	dept_name;
```



in/not in 型：子查询的结果是单列多行，作为 where 的过滤条件

```mysql
-- 带 IN 关键字的子查询
-- 查询出北京地区所有的员工信息
SELECT id FROM dept WHERE dept_location="北京";

SELECT * FROM worker WHERE dept_id IN (SELECT id FROM dept WHERE dept_location="北京");
SELECT * FROM worker WHERE dept_id NOT IN (SELECT id FROM dept WHERE dept_location="北京");
```



where 型：查询结果作为过滤条件出现在比较运算符的一端，常用于子查询结果为单个的情况。

```mysql
-- 查询出薪资大于公司平均薪资的员工 ID、姓名、薪资
SELECT AVG(salary) FROM worker;
SELECT id, name, salary FROM worker WHERE salary > (SELECT AVG(salary) FROM worker);
```



with...as...型：如果一整句查询语句中，某个子查询的结果会被多个父查询引用，通常建议将公用的子查询用简写表示出来。

```mysql
-- WITH [表名] AS (SELECT ...)
-- 查询出部门平均薪资大于公司平均薪资的部门名称、部门主管、所在地，及其平均薪资
WITH 
	dept_avg 
AS 
	(SELECT dept_id, AVG(salary) AS avg_salary FROM worker GROUP BY dept_id)
SELECT 
	dept_name, dept_manager, dept_location, avg_salary
FROM
	dept INNER JOIN dept_avg
ON
	id=dept_id
AND
	avg_salary > (SELECT AVG(avg_salary) FROM dept_avg);

```



#### 视图

视图是一种虚拟的表，它并不会在存储空间复制一份数据，而是对原有数据的一种引用。可以将视图理解为一种存储起来的 sql 语句。

- 视图可以简化多表查询
- 视图可以用于控制用户权限

````mysql
CREATE VIEW [视图名称] AS SELECT ...
````



