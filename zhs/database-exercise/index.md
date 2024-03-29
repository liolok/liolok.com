---
lang: zh-Hans
---

# 数据库原理题目整理

- 发布于 2018 年 5 月 19 日
- 更新于 2018 年 6 月 29 日

## 环境

- Microsoft SQL Server 2017 Express Edition
- Microsoft SQL Server Management Studio 17.6

## 目录

- [基本表的建立与修改](#基本表的建立与修改)
- [SELECT 语句的基本使用](#select-语句的基本使用)
- [SELECT 语句的嵌套使用](#select-语句的嵌套使用)
- [SQL 的存储操作](#sql-的存储操作)
- [视图的建立及操作](#视图的建立及操作)
- [创建触发器和存储过程](#创建触发器和存储过程)
- [查询语句及关系代数](#查询语句及关系代数)
- [参考资料](#参考资料)

## 基本表的建立与修改

成绩管理数据库 `GradeManager` 包括四个表：学生表 `Student`；课程表 `Course`；班级表 `Class`；成绩表 `Grade`。

### 学生表 Student

#### 表结构定义

| 属性名 | 数据类型    | 可否为空 | 含义         |
| ------ | ----------- | -------- | ------------ |
| Sno    | char(7)     | 否       | 学号（唯一） |
| Sname  | varchar(20) | 否       | 学生姓名     |
| Ssex   | char(2)     | 否       | 性别         |
| Sage   | smallint    | 可       | 年龄         |
| Clno   | char(5)     | 否       | 所在班级     |

```sql
-- 建立学生表
CREATE TABLE Student(
  Sno char(7) NOT NULL UNIQUE,
  Sname varchar(20) NOT NULL,
  Ssex char(2) NOT NULL,
  Sage smallint,
  Clno char(5) NOT NULL);
```

#### 表数据内容

| Sno     | Sname  | Ssex | Sage | Clno  |
| ------- | ------ | ---- | ---- | ----- |
| 2000101 | 李勇   | 男   | 20   | 00311 |
| 2000102 | 刘诗晨 | 女   | 19   | 00311 |
| 2000103 | 王一鸣 | 男   | 20   | 00312 |
| 2000104 | 张婷婷 | 女   | 21   | 00312 |
| 2001101 | 李勇敏 | 女   | 19   | 01311 |
| 2001102 | 贾向东 | 男   | 22   | 01311 |
| 2001103 | 陈宝玉 | 男   | 20   | 01311 |
| 2001104 | 张逸凡 | 男   | 21   | 01311 |

```sql
-- 填充学生表
INSERT INTO Student VALUES('2000101', '李勇', '男', 20, '00311');
INSERT INTO Student VALUES('2000102', '刘诗晨', '女', 19, '00311');
INSERT INTO Student VALUES('2000103', '王一鸣', '男', 20, '00312');
INSERT INTO Student VALUES('2000104', '张婷婷', '女', 21, '00312');
INSERT INTO Student VALUES('2001101', '李勇敏', '女', 19, '01311');
INSERT INTO Student VALUES('2001102', '贾向东', '男', 22, '01311');
INSERT INTO Student VALUES('2001103', '陈宝玉', '男', 20, '01311');
INSERT INTO Student VALUES('2001104', '张逸凡', '男', 21, '01311');
```

### 课程表 Couse

#### 表结构定义

| 属性名 | 数据类型    | 可否为空 | 含义           |
| ------ | ----------- | -------- | -------------- |
| Cno    | char(1)     | 否       | 课程号（唯一） |
| Cname  | varchar(20) | 否       | 课程名称       |
| Credit | smallint    | 可       | 学分           |

```sql
-- 建立课程表
CREATE TABLE Course(
  Cno char(1) NOT NULL UNIQUE,
  Cname varchar(20) NOT NULL,
  Credit smallint);
```

#### 表数据内容

| Cno | Cnmae        | Credit |
| --- | ------------ | ------ |
| 1   | 数据库       | 4      |
| 2   | 离散数学     | 3      |
| 3   | 管理信息系统 | 2      |
| 4   | 操作系统     | 4      |
| 5   | 数据结构     | 4      |
| 6   | 数据处理     | 2      |
| 7   | C语言        | 4      |

```sql
-- 填充课程表
INSERT INTO Course VALUES('1', '数据库', 4);
INSERT INTO Course VALUES('2', '离散数学', 3);
INSERT INTO Course VALUES('3', '管理信息系统', 2);
INSERT INTO Course VALUES('4', '操作系统', 4);
INSERT INTO Course VALUES('5', '数据结构', 4);
INSERT INTO Course VALUES('6', '数据处理', 2);
INSERT INTO Course VALUES('7', 'C语言', 4);
```

### 班级表 CLass

#### 表结构定义

| 属性名     | 数据类型    | 可否为空 | 含义           |
| ---------- | ----------- | -------- | -------------- |
| Clno       | char(5)     | 否       | 班级号（唯一） |
| Speciality | varchar(20) | 否       | 班级所在专业   |
| Inyear     | char(4)     | 否       | 入校年份       |
| Number     | int         | 可       | 班级人数       |
| Monitor    | char(7)     | 可       | 班长学号       |

```sql
-- 建立班级表
CREATE TABLE Class(
  Clno char(5) NOT NULL UNIQUE,
  Speciality varchar(20) NOT NULL,
  Inyear char(4) NOT NULL,
  Number int,
  Monitor char(7));
```

#### 表数据内容

| Clno  | Speciality | Inyear | Number | Monitor |
| ----- | ---------- | ------ | ------ | ------- |
| 00311 | 计算机软件 | 2000   | 120    | 2000101 |
| 00312 | 计算机应用 | 2000   | 140    | 2000103 |
| 01311 | 计算机软件 | 2001   | 220    | 2001103 |

```sql
-- 填充班级表
INSERT INTO Class VALUES('00311', '计算机软件', '2000', 120, '2000101');
INSERT INTO Class VALUES('00312', '计算机应用', '2000', 140, '2000103');
INSERT INTO Class VALUES('01311', '计算机软件', '2001', 220, '2001103');
```

### 成绩表 Grade

#### 表结构定义

| 属性名 | 数据类型      | 可否为空 | 含义   |
| ------ | ------------- | -------- | ------ |
| Sno    | char(7)       | 否       | 学号   |
| Cno    | char(1)       | 否       | 课程号 |
| Gmark  | numeric(4, 1) | 可       | 成绩   |

```sql
-- 建立成绩表
CREATE TABLE Grade(
  Sno char(7) NOT NULL,
  Cno char(1) NOT NULL,
  Gmark numeric(4, 1));
```

#### 表数据内容

| Sno     | Cno | Gmark |
| ------- | --- | ----- |
| 2000101 | 1   | 92    |
| 2000101 | 3   | NULL  |
| 2000101 | 5   | 86    |
| 2000102 | 1   | 78    |
| 2000102 | 6   | 55    |
| 2000103 | 3   | 65    |
| 2000103 | 6   | 78    |
| 2000103 | 5   | 66    |
| 2000104 | 1   | 54    |
| 2000104 | 6   | 83    |
| 2001101 | 2   | 70    |
| 2001101 | 4   | 65    |
| 2001102 | 2   | 80    |
| 2001102 | 4   | NULL  |
| 2001103 | 1   | 83    |
| 2001103 | 2   | 76    |
| 2001103 | 4   | 56    |
| 2001103 | 7   | 88    |

```sql
-- 填充成绩表
INSERT INTO Grade VALUES('2000101', '1', 92);
INSERT INTO Grade VALUES('2000101', '3', NULL);
INSERT INTO Grade VALUES('2000101', '5', 86);
INSERT INTO Grade VALUES('2000102', '1', 78);
INSERT INTO Grade VALUES('2000102', '6', 55);
INSERT INTO Grade VALUES('2000103', '3', 65);
INSERT INTO Grade VALUES('2000103', '6', 78);
INSERT INTO Grade VALUES('2000103', '5', 66);
INSERT INTO Grade VALUES('2000104', '1', 54);
INSERT INTO Grade VALUES('2000104', '6', 83);
INSERT INTO Grade VALUES('2001101', '2', 70);
INSERT INTO Grade VALUES('2001101', '4', 65);
INSERT INTO Grade VALUES('2001102', '2', 80);
INSERT INTO Grade VALUES('2001102', '4', NULL);
INSERT INTO Grade VALUES('2001103', '1', 83);
INSERT INTO Grade VALUES('2001103', '2', 76);
INSERT INTO Grade VALUES('2001103', '4', 56);
INSERT INTO Grade VALUES('2001103', '7', 88);
```

### 基本表的修改

```sql
-- 习题 3.11

-- (1) 给学生表增加一属性 Nation（民族），数据类型为 Varchar(20)；
ALTER TABLE Student
  ADD Nation varchar(20);

-- (2) 删除学生表中新增的属性 Nation；
ALTER TABLE Student
  DROP COLUMN Nation;

-- (3) 向成绩表中插入记录 ('2001110', '3', 80)；
INSERT INTO Grade(Sno, Cno, Gmark)
  VALUES('2001110', '3', 80);

-- (4) 将学号为“2001110”的学生成绩修改为 70 分；
UPDATE Grade
  SET Gmark = 70
  WHERE Sno = '2001110';

-- (5) 删除学号为“2001110”的学生的成绩记录；
DELETE FROM Grade
  WHERE Sno = '2001110';

-- (6) 在学生表的 Clno 属性上创建一个名为 IX_Class 的索引，以班级号的升序排序；
CREATE INDEX IX_Class ON Student (Clno);

-- (7) 删除学生表上的 IX_Class 索引。
DROP INDEX Student.IX_Class;
```

## SELECT 语句的基本使用

```sql
-- 习题 3.12

-- (1) 找出所有被学生选修了的课程号；
SELECT DISTINCT Cno FROM Grade;

-- (2) 找出 01311 班女学生的个人信息；
SELECT * FROM Student WHERE Clno = '01311' AND Ssex = '女';

-- (3) 找出 01311 班和 01312 班的学生姓名，性别，出生年份；
SELECT Sname, Ssex, (2018 - Sage) AS BirthYear
  FROM Student WHERE Clno IN ('01311', '01312');

-- (4) 找出所有姓李的学生的个人信息；
SELECT * FROM Student WHERE Sname LIKE '李%';

-- (5) 找出学生李勇所在班级的学生人数；
--- 写法1：查询与李勇班级号相同的 Student 即学生元组个数
SELECT COUNT(Sno) FROM Student
  WHERE Clno = (SELECT Clno FROM Student WHERE Sname = '李勇');
--- 写法2：查询 Class 的班级人数属性 Number
SELECT Number FROM Class C, Student S
  WHERE C.Clno = S.Clno AND Sname = '李勇';
--- 两种写法的区别在于如何理解题目要求的“学生人数”：前者是数据库中的学生数据个数，
--- 后者是班级的人数数据，显然是题目设计缺陷导致了两者的一致性无法得到保证。

-- (6) 找出名为操作系统的课程平均成绩，最高分，最低分；
SELECT AVG(Gmark), MAX(Gmark), MIN(Gmark)
  FROM Course C, Grade G
  WHERE C.Cno = G.Cno AND Cname = '操作系统';

-- (7) 找出选修了课程的学生人数；
SELECT COUNT(DISTINCT Sno) FROM Grade;

-- (8) 找出选修了课程操作系统的学生人数；
SELECT COUNT(DISTINCT Sno)
  FROM Course C, Grade G
  WHERE C.Cno = G.Cno AND Cname = '操作系统';

-- (9) 找出 2000 级计算机软件班的成绩为空的学生姓名。
SELECT DISTINCT Sname
  FROM Student S, Class C, Grade G
  WHERE S.Clno = C.Clno AND S.Sno = G.Sno
    AND Inyear = '2000' AND Speciality = '计算机软件'
    AND Gmark IS NULL;
```

## SELECT 语句的嵌套使用

```sql
-- 习题 3.13

-- (1) 找出与李勇在同一个班级的学生信息；
SELECT * FROM Student
  WHERE Clno = (SELECT Clno FROM Student WHERE Sname = '李勇')
    AND Sname <> '李勇';

-- (2) 找出所有与李勇有相同选修课程的学生信息；
SELECT * FROM Student
  WHERE Sno IN (SELECT Sno FROM Grade
    WHERE Cno IN (SELECT Cno FROM Student S, Grade G
      WHERE S.Sno = G.Sno AND S.Sname = '李勇'))
  AND Sname <> '李勇';

-- (3) 找出年龄介于学生李勇和 25 岁之间的学生信息（已知李勇年龄小于 25 岁）；
SELECT * FROM Student
  WHERE Sage BETWEEN (SELECT Sage FROM Student WHERE Sname = '李勇') AND 25;
--- BETWEEN...AND...查询结果包含范围左右边界

-- (4) 找出选修了课程操作系统的学生学号和姓名；
SELECT S.Sno, Sname FROM Student S, Course C, Grade G
  WHERE S.Sno = G.Sno AND C.Cno = G.Cno AND Cname = '操作系统';

-- (5) 找出没有选修1号课程的学生姓名；
SELECT Sname FROM Student
  WHERE Sno NOT IN (SELECT Sno FROM Grade WHERE Cno = 1);
--- 相关子查询
SELECT Sname FROM Student S
  WHERE NOT EXISTS (SELECT Sno FROM Grade
    WHERE S.Sno = Sno AND Cno = 1);

-- (6) 找出选修了全部课程的学生姓名。
SELECT Sname FROM Student S
  WHERE NOT EXISTS (SELECT * FROM Course C
    WHERE NOT EXISTS (SELECT * FROM Grade G
      WHERE G.Cno = C.Cno AND G.Sno = S.Sno));
--- （找出没有一门课程是其未选修的学生姓名）
```

```sql
-- 习题 3.14

-- (1) 查询选修了 3 号课程的学生学号及其成绩，并按成绩降序排列；
SELECT Sno, Gmark FROM Grade
  WHERE Cno = 3
  ORDER BY Gmark DESC;

-- (2) 查询全体学生信息，查询结果按班级号升序排列，同一班级学生按年龄降序排列；
SELECT * FROM Student
  ORDER BY Clno, Sage DESC;

-- (3) 求每个课程号及相应的选课人数；
SELECT Cno, COUNT(Sno) FROM Grade
  GROUP BY Cno;

-- (4) 查询选修了 3 门以上课程的学生学号。
SELECT Sno FROM Grade
  GROUP BY Sno
  HAVING COUNT(Cno) > 3;
```

## SQL 的存储操作

```sql
-- 习题 3.15

-- (1) 将 01311 班的全体学生的成绩置零；
UPDATE Grade
  SET COLUMN Gmark = 0
  WHERE Sno IN (SELECT Sno FROM Student WHERE Clno = '01311');

-- (2) 删除 2001 级计算机软件的全体学生的选课记录；
DELETE FROM Grade
  WHERE Sno IN (SELECT S.Sno FROM Student S, Class C
    WHERE S.Clno = C.Clno AND Speciality = '计算机软件');

-- (3) 学生李勇已退学，从数据库中删除有关他的记录；
UPDATE Class -- 如果李勇是某班班长（Monitor），则将该班 Monitor 属性置空以保证参照完整性。
  SET Monitor = NULL
  WHERE Monitor = (SELECT Sno FROM Student WHERE Sname = '李勇');
UPDATE Class -- 更新李勇所在班级人数
  SET Number = Number - 1
  WHERE Clno = (SELECT Clno FROM Student WHERE Sname = '李勇');
DELETE FROM Grade -- 先删引用
  WHERE Sno = (SELECT Sno FROM Student WHERE Sname = '李勇');
DELETE FROM Student -- 再删本体
  WHERE Sname = '李勇';

-- (4) 对每个班，求学生的平均年龄，并把结果存入数据库。
--- 写法1：建立视图存放查询结果
CREATE VIEW Cl_avg_age
  AS SELECT Clno, AVG(Sage) AS avg_age FROM Student;
--- 写法2：在原 Class 表中增加一列属性存放查询结果
ALTER TABLE Class
  ADD avg_age smallint NULL;
UPDATE Class
  SET avg_age = (SELECT AVG(Sage) FROM Student
    GROUP BY Clno HAVING Clno = Class.Clno);
```

## 视图的建立及操作

```sql
-- 习题 3.16

-- (1) 建立 01311 班选修了 1 号课程的学生视图 Stu_01311_1；
CREATE VIEW Stu_01311_1
  AS SELECT * FROM Student
    WHERE Sno IN (SELECT S.Sno
      FROM Student S, Grade G
      WHERE S.Sno = G.Sno
        AND S.Clno = '01311'
        AND G.Cno = 1);

-- (2) 建立 01311 班选修了 1 号课程并且成绩不及格的学生视图 Stu_01311_2；
CREATE VIEW Stu_01311_2
  AS SELECT * FROM Student
    WHERE Sno IN (SELECT S.Sno
      FROM Student S, Grade G
      WHERE S.Sno = G.Sno
        AND S.Clno = '01311'
        AND G.Cno = 1
        AND Gmark < 60);

-- (3) 建立视图 Stu_year，由学生学号，姓名，出生年份组成；
CREATE VIEW Stu_year(Sno, Sname, BirthYear)
  AS SELECT Sno, Sname, (2018 - Sage) FROM Student;

-- (4) 查询 1990 年以后出生的学生姓名；
SELECT Sname FROM Stu_year WHERE BirthYear > 1990;

-- (5) 查询 01311 班选修了 1 号课程并且成绩不及格的学生的学号，姓名，出生年份；
SELECT * FROM Stu_year
  WHERE Sno IN (SELECT Sno FROM Stu_01311_2);
```

## 创建触发器和存储过程

```sql
-- 在查询分析器（Query Analyzer）中创建以下触发器，并验证其语法的正确性：

-- 1.
-- 为 Student 表创建一插入和更新触发器 tri_ins_upd_student：
-- 当插入新的学生或者更新学生所在班级号时，检查该班级的学生人数有没有超过 40 人，
-- 如果没有则插入或者更新成功，如果超出 40 人，操作回滚。
CREATE TRIGGER tri_ins_upd_student
  ON Student
  AFTER INSERT, UPDATE
AS
  IF 40 < (SELECT Number FROM Class WHERE Clno = (SELECT Clno FROM inserted))
    ROLLBACK TRANSACTION

-- 2.
-- 为数据库 GradeManager 创建一存储过程 ap_returncount：
-- 功能：输入学生学号，输出该学生所在的班级人数。
CREATE PROCEDURE ap_returncount
  @Sno char(7)
AS
  DECLARE @count int
  SELECT @count = Number FROM Class
    WHERE Clno = (SELECT Clno FROM Student WHERE Sno = @Sno)
  RETURN @count

-- 3.
-- 在 Enterprise Manager 中展开 GradeManager 数据库，展开 Trigger，
-- 查看刚创建的触发器 tri_ins_upd_student，修改触发器定义，
-- 使之调用存储过程 ap_returncount 实现原来的功能。
CREATE TRIGGER tri_ins_upd_student
  ON Student
  AFTER INSERT, UPDATE
AS
  DECLARE @count int
  EXECUTE @count = ap_returncount (SELECT Sno FROM inserted)
  IF 40 < @count
    ROLLBACK TRANSACTION
```

## 查询语句及关系代数

在“学生——选课——课程”数据库中的 3 个关系如下：

- S (Sno, Sname, Age, Sex, Class);
- SC (Sno, Cno, Grade);
- C (Cno, Cname, Period, Teacher);

其中：

- `S` 是学生关系，`Sno`：学号，`Sname`：姓名，`Age`：年龄，`Sex`：性别，`Class`：班级；
- `SC` 是学生选课关系，`Sno`：学号，`Cno`：课程号，`Grade`：成绩；
- `C` 是课程关系，`Cno`：课程号，`Cname`：课程名，`Period`：学时，`Teacher`：任课教师。

```sql
-- 查询选修了课程名为 DB 的学生姓名和所在班级
-- ΠSname,Class(S∞SC∞σCname='DB'(C))
SELECT Sname, Class FROM S
  WHERE Sno IN (SELECT Sno FROM SC, C
    WHERE SC.Cno = C.Cno AND Cname = 'DB');
```

```sql
-- 查询 LIU 老师所授课程的课程号和课程名
-- ΠCno,Cname(σTeacher='LIU'(C))
SELECT Cno, Cname FROM C WHERE Teacher = 'LIU';
```

```sql
-- 查询学号为 S3 学生所学课程的课程名与任课教师名
-- ΠCname,Teacher(σSno='S3'(S∞SC))
SELECT Cname, Teacher FROM SC, C
  WHERE SC.Cno = C.Cno AND Sno = 'S3';
```

```sql
-- 查询至少选修 LIU 老师所授课程中一门课程的女学生的姓名
-- ΠSname(σSex='女'∧Teacher='LIU'(S∞SC∞C))
SELECT Sname FROM S
  WHERE Sex = '女'
    AND Sno IN (SELECT Sno FROM SC, C
      WHERE SC.Cno = C.Cno AND Teacher = 'LIU'));
```

```sql
-- 查询 WANG 同学不学的课程号
-- ΠCno(C)-ΠCno(σSname='WANG'(S∞SC))
SELECT Cno FROM C
  WHERE Cno NOT IN (SELECT Cno FROM S, SC
    WHERE S.Sno = SC.Sno AND Sname = 'WANG');
```

```sql
-- 查询至少选修两门课程的学生学号
-- ΠSno(σ1=4∧2≠5(SC×SC))
-- （SC 自乘之后，同一个学号（1、4 列）下两个课程号（2、5 列）不同的元组）
SELECT Sno FROM SC GROUP BY Sno HAVING COUNT(Cno) >＝ 2;
```

```sql
-- 查询全部学生都选修的课程的课程号和课程名
-- ΠCno,Cname(SC∞(ΠSno,Cno(SC)÷ΠSno(S)))
-- （涉及到全部值时，应用除法，“除数”是全部量）
SELECT Cno, Cname FROM C
  WHERE NOT EXISTS (SELECT * FROM S
    WHERE NOT EXISTS (SELECT * FROM SC
      WHERE S.Sno = SC.Sno AND SC.Cno = C.Cno));
```

```sql
-- 查询选修课程包含 LIU 老师所授课程的学生学号
-- ΠSno(σTeacher='LIU'(SC∞C))
SELECT Sno FROM SC, C
  WHERE SC.Cno = C.Cno AND Teacher = 'LIU'));
```

```sql
-- 统计有学生选修的课程门数
SELECT COUNT(DISTINCT Cno) FROM SC;
```

```sql
-- 求选修 C4 课程的学生的平均年龄
SELECT AVG(Age) FROM S
  WHERE Sno IN (SELECT Sno FROM SC WHERE Cno = 'C4');
```

```sql
-- 求 LIU 老师所授课程的每门课程的学生平均成绩
SELECT Cno, AVG(Grade) FROM SC
  WHERE Cno IN (SELECT Cno FROM C WHERE Teacher = 'LIU')
  GROUP BY Cno;
```

```sql
-- 统计每门课程的学生选修人数（超过 10 人的课程才统计）
-- 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
SELECT DISTINCT Cno, COUNT(Sno) FROM SC
  GROUP BY Cno
  HAVING COUNT(Sno) > 10
  ORDER BY COUNT(Sno) DESC, Cno;
```

```sql
-- 查询学号比 WANG 同学大，而年龄比他小的学生姓名
SELECT X.Sname FROM S X, S Y
  WHERE X.Sno > Y.Sno AND X.Age < Y.Age AND Y.Sname = 'WANG';
```

```sql
-- 查询姓名以 WANG 打头的所有学生的姓名和年龄
SELECT Sname, Age FROM S WHERE Sname LIKE 'WANG%';
```

```sql
-- 在 SC 中检索成绩为空值的学生学号和课程号
SELECT Sno, Cno FROM SC WHERE Grade IS NULL;
```

```sql
-- 求年龄大于女同学平均年龄的男学生姓名和年龄
SELECT Sname, Age FROM S X
  WHERE X.Sex = '男'
    AND X.Age > (SELECT AVG(Age) FROM S Y WHERE Y.Sex = '女');
```

```sql
-- 求年龄大于所有女同学年龄的男学生姓名和年龄
SELECT Sname, Age FROM S X
  WHERE X.Sex = '男'
    AND X.Age > ALL (SELECT Age FROM S Y WHERE Y.Sex = '女');
```

## 参考资料

教材：[《数据库原理(第四版)》 - 亚马逊](https://www.amazon.cn/dp/B01KUN09NE)
