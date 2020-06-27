# SQL语句的执行顺序

## 语法

? select

```mysql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr] ...
    [into_option]
    [FROM table_references
      [PARTITION partition_list]]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [PROCEDURE procedure_name(argument_list)]
    [into_option]
    [FOR UPDATE | LOCK IN SHARE MODE]

into_option: {
    INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
  | INTO DUMPFILE 'file_name'
  | INTO var_name [, var_name] ...
}
```

## 执行顺序

1. from
2. on
   - 先on过滤条件之后，才会join生成临时表
3. join
4. where
   - 临时表生成之后，根据限制条件从临时表中筛选
   - 在分组（聚集函数）之前筛选数据
5. group by
   - 分组之后，执行聚集函数
6. having
   - 聚合函数执行之后对分组数据进一步筛选，同group by一起使用，不可单独使用
7. select
   - 如果有group by使用的话，select查询的字段可能是group by后跟的分组字段，也有可能是对字段聚合函数计算的结果
8. distinct
9. order by
   - group by和orderby可以实现组内排序，即 `group by A,B order by A,B`
10. limit



# 初始化数据

```mysql
create database school charset=utf8;
use school;

DROP TABLE IF EXISTS `student`;
CREATE TABLE IF NOT EXISTS `student`(
`s_id` VARCHAR(20),
`s_name` VARCHAR(20) NOT NULL DEFAULT '',
`s_birth` VARCHAR(20) NOT NULL DEFAULT '',
`s_sex` VARCHAR(10) NOT NULL DEFAULT '',
PRIMARY KEY(`s_id`)
);
insert into student values('01' , '赵雷' , '1990-01-01' , '男');
insert into student values('02' , '钱电' , '1990-12-21' , '男');
insert into student values('03' , '孙风' , '1990-05-20' , '男');
insert into student values('04' , '李云' , '1990-08-06' , '男');
insert into student values('05' , '周梅' , '1991-12-01' , '女');
insert into student values('06' , '吴兰' , '1992-03-01' , '女');
insert into student values('07' , '郑竹' , '1989-07-01' , '女');
insert into student values('08' , '王菊' , '1990-01-20' , '女');

DROP TABLE IF EXISTS `teacher`;
CREATE TABLE IF NOT EXISTS `teacher`(
`t_id` VARCHAR(20),
`t_name` VARCHAR(20) NOT NULL DEFAULT '',
PRIMARY KEY(`t_id`)
);
insert into teacher values('01' , '张三');
insert into teacher values('02' , '李四');
insert into teacher values('03' , '王五');

DROP TABLE IF EXISTS `course`;
CREATE TABLE IF NOT EXISTS `course`(
`c_id` VARCHAR(20),
`c_name` VARCHAR(20) NOT NULL DEFAULT '',
`t_id` VARCHAR(20) NOT NULL,
PRIMARY KEY(`c_id`)
);
insert into course values('01' , '语文' , '02');
insert into course values('02' , '数学' , '01');
insert into course values('03' , '英语' , '03');

DROP TABLE IF EXISTS `score`;
CREATE TABLE IF NOT EXISTS `score`(
`s_id` VARCHAR(20),
`c_id` VARCHAR(20),
`s_score` INT(3),
PRIMARY KEY(`s_id`,`c_id`)
);
insert into score values('01' , '01' , 80);
insert into score values('01' , '02' , 90);
insert into score values('01' , '03' , 99);
insert into score values('02' , '01' , 70);
insert into score values('02' , '02' , 60);
insert into score values('02' , '03' , 80);
insert into score values('03' , '01' , 80);
insert into score values('03' , '02' , 80);
insert into score values('03' , '03' , 80);
insert into score values('04' , '01' , 50);
insert into score values('04' , '02' , 30);
insert into score values('04' , '03' , 20);
insert into score values('05' , '01' , 76);
insert into score values('05' , '02' , 87);
insert into score values('06' , '01' , 31);
insert into score values('06' , '03' , 34);
insert into score values('07' , '02' , 89);
insert into score values('07' , '03' , 98);
```



# 解题

## 一 查询"01"课程比"02"课程成绩高的学生的信息及课程分数

```mysql
select s1.s_id as s_id,s1.s_score as c1,s2.s_score as c2,student.s_name,student.s_birth,student.s_sex from
(select * from score where c_id='01') s1
inner join 
(select * from score where c_id='02') s2
on s1.s_id=s2.s_id and s1.s_score>s2.s_score
inner join
student
on s1.s_id=student.s_id;

+------+------+------+--------+------------+-------+
| s_id | c1   | c2   | s_name | s_birth    | s_sex |
+------+------+------+--------+------------+-------+
| 02   |   70 |   60 | 钱电   | 1990-12-21 | 男    |
| 04   |   50 |   30 | 李云   | 1990-08-06 | 男    |
+------+------+------+--------+------------+-------+
```

- 在比较01课程大于02课程时，可以用on过滤，再去join；也可以join生成临时表之后，再用where过滤。很显然，on优于where。



## 二 查询"01"课程比"02"课程成绩低的学生的信息及课程分数

```mysql
select student.*,s1.s_score c1,s2.s_score c2 from student
inner join (select * from score where c_id='01') s1 on student.s_id=s1.s_id
inner join (select * from score where c_id='02') s2 on s1.s_id=s2.s_id
where s1.s_score < s2.s_score;

+------+--------+------------+-------+------+------+
| s_id | s_name | s_birth    | s_sex | c1   | c2   |
+------+--------+------------+-------+------+------+
| 01   | 赵雷   | 1990-01-01 | 男    |   80 |   90 |
| 05   | 周梅   | 1991-12-01 | 女    |   76 |   87 |
+------+--------+------------+-------+------+------+
```



## 三 查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

```mysql
select student.s_id,student.s_name,avgs from 
(select s_id,avg(s_score) as avgs from score group by s_id having avgs>=60) s1
inner join student
on s1.s_id=student.s_id;

+------+--------+---------+
| s_id | s_name | avgs    |
+------+--------+---------+
| 01   | 赵雷   | 89.6667 |
| 02   | 钱电   | 70.0000 |
| 03   | 孙风   | 80.0000 |
| 05   | 周梅   | 81.5000 |
| 07   | 郑竹   | 93.5000 |
+------+--------+---------+
```



## 四 查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
```mysql
select student.s_id,student.s_name,avgs from 
(select s_id,avg(s_score) as avgs from score group by s_id having avgs<60) s1
inner join student
on s1.s_id=student.s_id
union 
select student.s_id,student.s_name,0 from student left join score on student.s_id=score.s_id where s_score is null;

+------+--------+---------+
| s_id | s_name | avgs    |
+------+--------+---------+
| 04   | 李云   | 33.3333 |
| 06   | 吴兰   | 32.5000 |
| 08   | 王菊   |  0.0000 |
+------+--------+---------+
```

- 包括没有考试的学生，其平均成绩是0。



## 五 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩

```mysql
select student.s_id,student.s_name,
case when s.sumcourse is null then 0 else s.sumcourse end as sumcourse,
case when s.sumscore is null then 0 else s.sumscore end as sumscore
from student
left join
(select s_id, count(*) as sumcourse, sum(s_score) as sumscore from score group by s_id) s
on student.s_id=s.s_id;

+------+--------+-----------+----------+
| s_id | s_name | sumcourse | sumscore |
+------+--------+-----------+----------+
| 01   | 赵雷   |         3 |      269 |
| 02   | 钱电   |         3 |      210 |
| 03   | 孙风   |         3 |      240 |
| 04   | 李云   |         3 |      100 |
| 05   | 周梅   |         2 |      163 |
| 06   | 吴兰   |         2 |       65 |
| 07   | 郑竹   |         2 |      187 |
| 08   | 王菊   |         0 |        0 |
+------+--------+-----------+----------+
```

- 流程控制

  - case when

    ```mysql
    -- 表示等值比较（value和compare-value）；
    -- 如果没有匹配，则返回ELSE后的结果；
    -- 如果没有ELSE部分，则返回NULL
    CASE value 
    	WHEN [compare-value] THEN result 
    	[WHEN [compare-value] THEN result ...] 
    	[ELSE result] 
    END 
    
    -- 条件是否返回true（不为0和NULL），成立则执行后边的语句，然后退出；
    	-- 对于多个when条件同时成立，只会执行第一个满足条件的when
    -- 如果都不为true，则返回ELSE后的结果；
    -- 如果没有ELSE部分，则返回NULL
    CASE 
    	WHEN [condition] THEN result 
    	[WHEN [condition] THEN result ...]
    	[ELSE result] 
    END
    ```

  - if

    ```mysql
    -- 如果expr1为true（不为 0 和 NULL），则IF()的返回值为expr2；
    -- 否则，返回值为expr3 
    IF(expr1, expr2, expr3) 
    ```

    

## 六 查询"李"姓老师的数量

```mysql
select count(*) from teacher where t_name like '李%';

+----------+
| count(*) |
+----------+
|        1 |
+----------+
```



## 七 查询学过"张三"老师授课的同学的信息

```mysql
select * from student where s_id in (
select s.s_id from 
(select c_id from teacher t left join course c on t.t_id=c.t_id where t_name='张三') a left join score s on a.c_id=s.c_id
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 01   | 赵雷   | 1990-01-01 | 男    |
| 02   | 钱电   | 1990-12-21 | 男    |
| 03   | 孙风   | 1990-05-20 | 男    |
| 04   | 李云   | 1990-08-06 | 男    |
| 05   | 周梅   | 1991-12-01 | 女    |
| 07   | 郑竹   | 1989-07-01 | 女    |
+------+--------+------------+-------+
```



## 八 查询没学过"张三"老师授课的同学的信息

```mysql
select * from student where s_id not in (
select s.s_id from 
(select c_id from teacher t left join course c on t.t_id=c.t_id where t_name='张三') a left join score s on a.c_id=s.c_id
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 06   | 吴兰   | 1992-03-01 | 女    |
| 08   | 王菊   | 1990-01-20 | 女    |
+------+--------+------------+-------+
```



## 九 查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息

```mysql
select * from student where s_id in (
select a.s_id from 
(select s_id from score where c_id='01') a
inner join
(select s_id from score where c_id='02') b
on a.s_id =b.s_id
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 01   | 赵雷   | 1990-01-01 | 男    |
| 02   | 钱电   | 1990-12-21 | 男    |
| 03   | 孙风   | 1990-05-20 | 男    |
| 04   | 李云   | 1990-08-06 | 男    |
| 05   | 周梅   | 1991-12-01 | 女    |
+------+--------+------------+-------+
```



## 十 查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息

```mysql
select * from student where s_id in (
select a.s_id from 
(select s_id from score where c_id='01') a
inner join
(select s_id from student where s_id not in (select s_id from score where c_id='02')) b
on a.s_id =b.s_id
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 06   | 吴兰   | 1992-03-01 | 女    |
+------+--------+------------+-------+
```



## 十一 查询没有学全所有课程的同学的信息

```mysql
select * from student where s_id not in (
  select s_id from 
  (
    select s_id, count(*) cs from score group by s_id having cs=(select count(*) from course)
  ) a
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 05   | 周梅   | 1991-12-01 | 女    |
| 06   | 吴兰   | 1992-03-01 | 女    |
| 07   | 郑竹   | 1989-07-01 | 女    |
| 08   | 王菊   | 1990-01-20 | 女    |
+------+--------+------------+-------+
```



## 十二 查询至少有一门课与学号为"01"的同学所学相同的同学的信息

```mysql
select * from student where s_id in (
select distinct b.s_id from 
(select c_id from score where s_id='01') a
left join
(select s_id,c_id from score where s_id!='01') b
on a.c_id = b.c_id
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 02   | 钱电   | 1990-12-21 | 男    |
| 03   | 孙风   | 1990-05-20 | 男    |
| 04   | 李云   | 1990-08-06 | 男    |
| 05   | 周梅   | 1991-12-01 | 女    |
| 06   | 吴兰   | 1992-03-01 | 女    |
| 07   | 郑竹   | 1989-07-01 | 女    |
+------+--------+------------+-------+
```



## 十三 查询和"01"号的同学学习的课程完全相同的其他同学的信息

```mysql
select * from student where s_id in
(
	select b.s_id from 
		(select s_id,count(*) as mycourse from score group by s_id) a
	left join 
		(
			select b.s_id,count(*) as samecourse from
				(select c_id from score where s_id='01') a
			inner join 
				score b
			on a.c_id = b.c_id
			group by s_id
		) b
	on a.s_id=b.s_id
  where samecourse=(select count(*) from score where s_id='01') and samecourse=mycourse
) and s_id <> '01';

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 02   | 钱电   | 1990-12-21 | 男    |
| 03   | 孙风   | 1990-05-20 | 男    |
| 04   | 李云   | 1990-08-06 | 男    |
+------+--------+------------+-------+
```

- 当两个同学学习课程的数目和名称相同时，为"完全相同"
  - "01"号同学学习的课程是1、2和3，有A同学学习的课程是1、2和3，则"01"号同学和A同学学习的课程完全相同。
  - "01"号同学学习的课程是1和2，有A同学学习的课程是1、2和3，虽然"01"号同学学习的课程全被A同学学习，但A同学和"01"号同学学习的课程数量不同。



## 十四 查询没学过"张三"老师讲授的任一门课程的学生姓名

```mysql
select s_name from student where s_id not in (
	select distinct s_id from score where c_id in (
    select c_id from course where t_id = (select t_id from teacher where t_name='张三')
  )
);

+--------+
| s_name |
+--------+
| 吴兰   |
| 王菊   |
+--------+
```



## 十五 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```mysql
select a.s_id,a.s_name,avgscore from 
	(
    select s_id,s_name from student where s_id in (
      select s_id from score where s_score<60 group by s_id having count(s_id)>=2
    )
  ) a
left join
  (select s_id,avg(s_score) as avgscore from score group by s_id) b
on a.s_id=b.s_id;

+------+--------+----------+
| s_id | s_name | avgscore |
+------+--------+----------+
| 04   | 李云   |  33.3333 |
| 06   | 吴兰   |  32.5000 |
+------+--------+----------+
```



## 十六 检索"01"课程分数小于60，按分数降序排列的学生信息

```mysql
select * from student where s_id in (
  select s_id from score where c_id='01' and s_score<60 order by s_score desc
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 04   | 李云   | 1990-08-06 | 男    |
| 06   | 吴兰   | 1992-03-01 | 女    |
+------+--------+------------+-------+
```



## 十七 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

```mysql
select 
	student.s_id, 
	if(yw is null,0,yw) as '语文',
	if(sx is null,0,sx) as '数学',
	if(yy is null,0,yy) as '英语',
	if(pjcj is null,0,pjcj) as '平均成绩'
from student
left join
	(select s_id,s_score as yw from score where c_id='01') a
on student.s_id=a.s_id
left join
	(select s_id,s_score as sx from score where c_id='02') b
on student.s_id=b.s_id
left join
	(select s_id,s_score as yy from score where c_id='03') c
on student.s_id=c.s_id
left join
	(select s_id,avg(s_score) as pjcj from score group by s_id) d
on student.s_id=d.s_id
order by pjcj desc;

+------+--------+--------+--------+--------------+
| s_id | 语文   | 数学   | 英语   | 平均成绩     |
+------+--------+--------+--------+--------------+
| 07   |      0 |     89 |     98 |      93.5000 |
| 01   |     80 |     90 |     99 |      89.6667 |
| 05   |     76 |     87 |      0 |      81.5000 |
| 03   |     80 |     80 |     80 |      80.0000 |
| 02   |     70 |     60 |     80 |      70.0000 |
| 04   |     50 |     30 |     20 |      33.3333 |
| 06   |     31 |      0 |     34 |      32.5000 |
| 08   |      0 |      0 |      0 |       0.0000 |
+------+--------+--------+--------+--------------+
```



## 十八 查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name ，最高分，最低分，平均分，及格率，中等率，优良率，优秀率

```mysql
select 
	course.c_id,
	course.c_name,
	if(maxscore is null,0,maxscore) as '最高分',
	if(minscore is null,0,minscore) as '最低分',
	if(avgscore is null,0,avgscore) as '平均分',
  if((jg/totalcourse)*100 is null,0,(jg/totalcourse)*100) as '及格率',
  if((zd/totalcourse)*100 is null,0,(zd/totalcourse)*100) as '中等率',
	if((yl/totalcourse)*100 is null,0,(yl/totalcourse)*100) as '优良率',
	if((yx/totalcourse)*100 is null,0,(yx/totalcourse)*100) as '优秀率'
from course
left join
	(select c_id,max(s_score) as maxscore from score group by c_id ) a
on course.c_id=a.c_id
left join
	(select c_id,min(s_score) as minscore from score group by c_id ) b
on course.c_id=b.c_id
left join
	(select c_id,avg(s_score) as avgscore,count(s_score) as totalcourse from score group by c_id ) c
on course.c_id=c.c_id
left join
	(select c_id, count(s_score) as jg from score where s_score>=60 group by c_id) d
on course.c_id=d.c_id
left join
	(select c_id, count(s_score) as zd from score where s_score>=70 and s_score<80 group by c_id) e
on course.c_id=e.c_id
left join
	(select c_id, count(s_score) as yl from score where s_score>=80 and s_score<90 group by c_id) f
on course.c_id=f.c_id
left join
	(select c_id, count(s_score) as yx from score where s_score>=90 group by c_id) g
on course.c_id=g.c_id
order by course.c_id;

+------+--------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| c_id | c_name | 最高分    | 最低分    | 平均分    | 及格率    | 中等率    | 优良率    | 优秀率    |
+------+--------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| 01   | 语文   |        80 |        31 |   64.5000 |   66.6667 |   33.3333 |   33.3333 |    0.0000 |
| 02   | 数学   |        90 |        30 |   72.6667 |   83.3333 |    0.0000 |   50.0000 |   16.6667 |
| 03   | 英语   |        99 |        20 |   68.5000 |   66.6667 |    0.0000 |   33.3333 |   33.3333 |
+------+--------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
```

- 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90



## 十九 按各科成绩进行排序，并显示排名

```mysql
select
	c_id,s_id,s_score,row_num,rank,dense_rank
from (
  select
    a.*,
    if (@precid!=c_id,@rownum:=1,@rownum:=@rownum+1) as row_num,
  	(case 
  		when @rownum=1 then @rank:=1
     	when @prescore=s_score then @rank
     	else @rank:=@rownum
  	end) as rank,
  	(case
     	when @rownum=1 then @denserank:=1
     	when @prescore=s_score then @denserank
      else @denserank:=@denserank+1
     end) as dense_rank,
    @precid:=c_id,
  	@prescore:=s_score
  from 
    (select * from score order by c_id,s_score desc) a,
  	(select @precid:=null,@rownum:=0,@prescore:=null,@rank:=0,@denserank:=0) b
) a;

+------+------+---------+---------+------+------------+
| c_id | s_id | s_score | row_num | rank | dense_rank |
+------+------+---------+---------+------+------------+
| 01   | 01   |      80 |       1 |    1 |          1 |
| 01   | 03   |      80 |       2 |    1 |          1 |
| 01   | 05   |      76 |       3 |    3 |          2 |
| 01   | 02   |      70 |       4 |    4 |          3 |
| 01   | 04   |      50 |       5 |    5 |          4 |
| 01   | 06   |      31 |       6 |    6 |          5 |
| 02   | 01   |      90 |       1 |    1 |          1 |
| 02   | 07   |      89 |       2 |    2 |          2 |
| 02   | 05   |      87 |       3 |    3 |          3 |
| 02   | 03   |      80 |       4 |    4 |          4 |
| 02   | 02   |      60 |       5 |    5 |          5 |
| 02   | 04   |      30 |       6 |    6 |          6 |
| 03   | 01   |      99 |       1 |    1 |          1 |
| 03   | 07   |      98 |       2 |    2 |          2 |
| 03   | 03   |      80 |       3 |    3 |          3 |
| 03   | 02   |      80 |       4 |    3 |          3 |
| 03   | 06   |      34 |       5 |    5 |          4 |
| 03   | 04   |      20 |       6 |    6 |          5 |
+------+------+---------+---------+------+------------+
```

- row_num

  连续排名，即使相同的值，依旧按照连续数字进行排名。

- rank

  并列跳跃排名，并列即相同的值，相同的值保留重复名次，遇到下一个不同值时，跳跃到总共的排名。

- dense_rank

  并列连续排序，并列即相同的值，相同的值保留重复名次，遇到下一个不同值时，依然按照连续数字排名。



## 二十 查询学生的总成绩并进行排名

```mysql
select s_id,sumscore,row_num,rank
from (
	select 
    s_id,
  	sumscore,
    if(@presumscore is null,@rownum:=1,@rownum:=@rownum+1) as row_num,
  	(case
     	when @rownum=1 then @rank:=1
     	when @presumscore=sumscore then @rank
     	else @rank:=@rownum
     end) as rank,
    @presumscore:=sumscore
  from 
    (select s_id,sum(s_score) as sumscore from score group by s_id order by sumscore desc) a,
    (select @rownum:=0,@presumscore:=null,@rank:=0) b
) a;

+------+----------+---------+------+
| s_id | sumscore | row_num | rank |
+------+----------+---------+------+
| 01   |      269 |       1 | 1    |
| 03   |      240 |       2 | 2    |
| 02   |      210 |       3 | 3    |
| 07   |      187 |       4 | 4    |
| 05   |      163 |       5 | 5    |
| 04   |      100 |       6 | 6    |
| 06   |       65 |       7 | 7    |
+------+----------+---------+------+
```



## 二十一 查询不同老师所教不同课程平均分从高到低显示

```mysql
select t.t_id,t.t_name,s.c_id,avg(s.s_score) as avgscore from teacher t
left join course c on t.t_id=c.t_id
left join score s on c.c_id=s.c_id
group by s.c_id,t.t_id,t.t_name
order by avgscore desc;

+------+--------+------+----------+
| t_id | t_name | c_id | avgscore |
+------+--------+------+----------+
| 01   | 张三   | 02   |  72.6667 |
| 03   | 王五   | 03   |  68.5000 |
| 02   | 李四   | 01   |  64.5000 |
+------+--------+------+----------+
```



## 二十二 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩

```mysql
select s.*,a.s_score,a.c_id,a.dense_rank
from (
  select
    c_id,s_id,s_score,row_num,rank,dense_rank
  from (
    select
      a.*,
      if (@precid!=c_id,@rownum:=1,@rownum:=@rownum+1) as row_num,
      (case 
        when @rownum=1 then @rank:=1
        when @prescore=s_score then @rank
        else @rank:=@rownum
      end) as rank,
      (case
        when @rownum=1 then @denserank:=1
        when @prescore=s_score then @denserank
        else @denserank:=@denserank+1
       end) as dense_rank,
      @precid:=c_id,
      @prescore:=s_score
    from 
      (select * from score order by c_id,s_score desc) a,
      (select @precid:=null,@rownum:=0,@prescore:=null,@rank:=0,@denserank:=0) b
  ) a where dense_rank in (2,3)
) a left join student s on a.s_id=s.s_id;

+------+--------+------------+-------+---------+------+------------+
| s_id | s_name | s_birth    | s_sex | s_score | c_id | dense_rank |
+------+--------+------------+-------+---------+------+------------+
| 05   | 周梅   | 1991-12-01 | 女    |      76 | 01   |          2 |
| 02   | 钱电   | 1990-12-21 | 男    |      70 | 01   |          3 |
| 07   | 郑竹   | 1989-07-01 | 女    |      89 | 02   |          2 |
| 05   | 周梅   | 1991-12-01 | 女    |      87 | 02   |          3 |
| 07   | 郑竹   | 1989-07-01 | 女    |      98 | 03   |          2 |
| 03   | 孙风   | 1990-05-20 | 男    |      80 | 03   |          3 |
| 02   | 钱电   | 1990-12-21 | 男    |      80 | 03   |          3 |
+------+--------+------------+-------+---------+------+------------+
```



## 二十三 统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

```mysql
select
	a.c_id,
	c.c_name,
	sum(l1) as `[0-60]`,
	sum(l1)/count(*)*100 as `[0-60]%`,
	sum(l2) as `[70-60]`,
	sum(l2)/count(*)*100 as `[70-60]%`,
	sum(l3) as `[85-70]`,
	sum(l3)/count(*)*100 as `[85-70]%`,
	sum(l4) as `[100-85]`,
	sum(l4)/count(*)*100 as `[100-85]%`
from 
 (
	select 
    score.*,
    (case when s_score>=0 and s_score<=60 then 1 else 0 end) as l1,
    (case when s_score>60 and s_score<=70 then 1 else 0 end) as l2,
    (case when s_score>70 and s_score<=85 then 1 else 0 end) as l3,
    (case when s_score>85 and s_score<=100 then 1 else 0 end) as l4
  from score
 ) a 
left join course c on a.c_id=c.c_id
group by a.c_id;

+------+--------+--------+---------+---------+----------+---------+----------+----------+-----------+
| c_id | c_name | [0-60] | [0-60]% | [70-60] | [70-60]% | [85-70] | [85-70]% | [100-85] | [100-85]% |
+------+--------+--------+---------+---------+----------+---------+----------+----------+-----------+
| 01   | 语文   |      2 | 33.3333 |       1 |  16.6667 |       3 |  50.0000 |        0 |    0.0000 |
| 02   | 数学   |      2 | 33.3333 |       0 |   0.0000 |       1 |  16.6667 |        3 |   50.0000 |
| 03   | 英语   |      2 | 33.3333 |       0 |   0.0000 |       2 |  33.3333 |        2 |   33.3333 |
+------+--------+--------+---------+---------+----------+---------+----------+----------+-----------+
```



## 二十四 查询学生平均成绩及其名次

```mysql
select s_id,avgscore,row_num,rank
from (
	select 
    s_id,
  	avgscore,
    if(@preavgscore is null,@rownum:=1,@rownum:=@rownum+1) as row_num,
  	(case
     	when @rownum=1 then @rank:=1
     	when @preavgscore=avgscore then @rank
     	else @rank:=@rownum
     end) as rank,
    @preavgscore:=avgscore
  from 
		(select s_id,avg(s_score) as avgscore from score group by s_id order by avgscore desc) a,
		(select @rownum:=0,@preavgscore:=null,@rank:=0) b
) a;

+------+----------+---------+------+
| s_id | avgscore | row_num | rank |
+------+----------+---------+------+
| 07   |  93.5000 |       1 |    1 |
| 01   |  89.6667 |       2 |    2 |
| 05   |  81.5000 |       3 |    3 |
| 03   |  80.0000 |       4 |    4 |
| 02   |  70.0000 |       5 |    5 |
| 04   |  33.3333 |       6 |    6 |
| 06   |  32.5000 |       7 |    7 |
+------+----------+---------+------+
```



## 二十五 查询各科成绩前三名的记录

```mysql
select s.*,a.s_score,a.c_id,a.dense_rank
from (
  select
    c_id,s_id,s_score,row_num,rank,dense_rank
  from (
    select
      a.*,
      if (@precid!=c_id,@rownum:=1,@rownum:=@rownum+1) as row_num,
      (case 
        when @rownum=1 then @rank:=1
        when @prescore=s_score then @rank
        else @rank:=@rownum
      end) as rank,
      (case
        when @rownum=1 then @denserank:=1
        when @prescore=s_score then @denserank
        else @denserank:=@denserank+1
       end) as dense_rank,
      @precid:=c_id,
      @prescore:=s_score
    from 
      (select * from score order by c_id,s_score desc) a,
      (select @precid:=null,@rownum:=0,@prescore:=null,@rank:=0,@denserank:=0) b
  ) a where dense_rank in (1,2,3)
) a left join student s on a.s_id=s.s_id;

+------+--------+------------+-------+---------+------+------------+
| s_id | s_name | s_birth    | s_sex | s_score | c_id | dense_rank |
+------+--------+------------+-------+---------+------+------------+
| 01   | 赵雷   | 1990-01-01 | 男    |      80 | 01   |          1 |
| 03   | 孙风   | 1990-05-20 | 男    |      80 | 01   |          1 |
| 05   | 周梅   | 1991-12-01 | 女    |      76 | 01   |          2 |
| 02   | 钱电   | 1990-12-21 | 男    |      70 | 01   |          3 |
| 01   | 赵雷   | 1990-01-01 | 男    |      90 | 02   |          1 |
| 07   | 郑竹   | 1989-07-01 | 女    |      89 | 02   |          2 |
| 05   | 周梅   | 1991-12-01 | 女    |      87 | 02   |          3 |
| 01   | 赵雷   | 1990-01-01 | 男    |      99 | 03   |          1 |
| 07   | 郑竹   | 1989-07-01 | 女    |      98 | 03   |          2 |
| 03   | 孙风   | 1990-05-20 | 男    |      80 | 03   |          3 |
| 02   | 钱电   | 1990-12-21 | 男    |      80 | 03   |          3 |
+------+--------+------------+-------+---------+------+------------+
```



## 二十六 查询每门课程被选修的学生数

```mysql
select c_id,count(*) as sn from score group by c_id;

+------+----+
| c_id | sn |
+------+----+
| 01   |  6 |
| 02   |  6 |
| 03   |  6 |
+------+----+
```



## 二十七 查询出只有两门课程的全部学生的学号和姓名

```mysql
select a.s_id,b.s_name from
	(select s_id from score group by s_id having count(c_id)=2) a
left join 
	student b 
on a.s_id=b.s_id;

+------+--------+
| s_id | s_name |
+------+--------+
| 05   | 周梅   |
| 06   | 吴兰   |
| 07   | 郑竹   |
+------+--------+
```



## 二十八 查询男生、女生人数

```mysql
select s_sex,count(s_id) from student group by s_sex;

+-------+-------------+
| s_sex | count(s_id) |
+-------+-------------+
| 女    |           4 |
| 男    |           4 |
+-------+-------------+
```



## 二十九 查询名字中含有"风"字的学生信息

```mysql
select * from student where s_name like '%风%';

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 03   | 孙风   | 1990-05-20 | 男    |
+------+--------+------------+-------+
```



## 三十 查询同名同姓学生名单，并统计同名人数

```mysql
select s_name,number-1 as `同名人数` from
	(select s_name,count(*) number from student group by s_name) a;
	
+--------+--------------+
| s_name | 同名人数     |
+--------+--------------+
| 吴兰   |            0 |
| 周梅   |            0 |
| 孙风   |            0 |
| 李云   |            0 |
| 王菊   |            0 |
| 赵雷   |            0 |
| 郑竹   |            0 |
| 钱电   |            0 |
+--------+--------------+
```



## 三十一 查询1990年出生的学生名单

```mysql
select * from student where year(s_birth)='1990';

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 01   | 赵雷   | 1990-01-01 | 男    |
| 02   | 钱电   | 1990-12-21 | 男    |
| 03   | 孙风   | 1990-05-20 | 男    |
| 04   | 李云   | 1990-08-06 | 男    |
| 08   | 王菊   | 1990-01-20 | 女    |
+------+--------+------------+-------+
```



## 三十二 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

```mysql
select c_id,avg(s_score) avgscore from score group by c_id order by avgscore desc,c_id;

+------+----------+
| c_id | avgscore |
+------+----------+
| 02   |  72.6667 |
| 03   |  68.5000 |
| 01   |  64.5000 |
+------+----------+
```



## 三十三 查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩

```mysql
select a.s_id,b.s_name,avgscore from
	(select s_id,avg(s_score) avgscore from score group by s_id having avgscore>=85) a
left join 
	student b
on a.s_id=b.s_id;

+------+--------+----------+
| s_id | s_name | avgscore |
+------+--------+----------+
| 01   | 赵雷   |  89.6667 |
| 07   | 郑竹   |  93.5000 |
+------+--------+----------+
```



## 三十四 查询课程名称为"数学"，且分数低于60的学生姓名和分数

```mysql
select student.s_name,score.s_score from 
	course
left join 
	score on course.c_id=score.c_id
left join 
	student on score.s_id=student.s_id
where course.c_name='数学' and score.s_score<60;

+--------+---------+
| s_name | s_score |
+--------+---------+
| 李云   |      30 |
+--------+---------+
```



## 三十五 查询所有学生的课程及分数情况

```mysql
select 
	student.s_id, 
	if(yw is null,0,yw) as '语文',
	if(sx is null,0,sx) as '数学',
	if(yy is null,0,yy) as '英语'
from student
left join
	(select s_id,s_score as yw from score where c_id='01') a
on student.s_id=a.s_id
left join
	(select s_id,s_score as sx from score where c_id='02') b
on student.s_id=b.s_id
left join
	(select s_id,s_score as yy from score where c_id='03') c
on student.s_id=c.s_id;

+------+--------+--------+--------+
| s_id | 语文   | 数学   | 英语   |
+------+--------+--------+--------+
| 01   |     80 |     90 |     99 |
| 02   |     70 |     60 |     80 |
| 03   |     80 |     80 |     80 |
| 04   |     50 |     30 |     20 |
| 05   |     76 |     87 |      0 |
| 06   |     31 |      0 |     34 |
| 07   |      0 |     89 |     98 |
| 08   |      0 |      0 |      0 |
+------+--------+--------+--------+
```



## 三十六 查询任何一门课程成绩在70分以上的姓名、课程名称和分数

```mysql
select b.s_name,c.c_name,a.s_score from 
	(select s_id,c_id,s_score from score where s_score>70) a
left join
	student b
on a.s_id=b.s_id
left join
	course c
on a.c_id=c.c_id;

+--------+--------+---------+
| s_name | c_name | s_score |
+--------+--------+---------+
| 赵雷   | 语文   |      80 |
| 孙风   | 语文   |      80 |
| 周梅   | 语文   |      76 |
| 赵雷   | 数学   |      90 |
| 孙风   | 数学   |      80 |
| 周梅   | 数学   |      87 |
| 郑竹   | 数学   |      89 |
| 赵雷   | 英语   |      99 |
| 钱电   | 英语   |      80 |
| 孙风   | 英语   |      80 |
| 郑竹   | 英语   |      98 |
+--------+--------+---------+
```



## 三十七 查询不及格的课程

```mysql
select a.s_id,a.c_id,b.c_name,a.s_score from
	(select s_id,c_id,s_score from score where s_score<60) a
left join 
	course b
on a.c_id=b.c_id;

+------+------+--------+---------+
| s_id | c_id | c_name | s_score |
+------+------+--------+---------+
| 04   | 01   | 语文   |      50 |
| 06   | 01   | 语文   |      31 |
| 04   | 02   | 数学   |      30 |
| 04   | 03   | 英语   |      20 |
| 06   | 03   | 英语   |      34 |
+------+------+--------+---------+
```



## 三十八 查询课程编号为01且课程成绩在80分以上的学生的学号和姓名

```mysql
select a.s_id,b.s_name from 
	(select s_id from score where c_id='01' and s_score>80) a
left join
	student b
on a.s_id=b.s_id;

Empty set
```



## 三十九 求每门课程的学生人数

```mysql
select c_id, count(*) from score group by c_id;

+------+----------+
| c_id | count(*) |
+------+----------+
| 01   |        6 |
| 02   |        6 |
| 03   |        6 |
+------+----------+
```



## 四十 查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩

```mysql
select b.*,a.c_id,a.s_score from 
(select s_id,c_id,s_score from score where c_id in (
  select c_id from course where t_id in
    (select t_id from teacher where t_name='张三'))
order by s_score desc 
limit 0,1) a
left join student b
on a.s_id=b.s_id;

+------+--------+------------+-------+------+---------+
| s_id | s_name | s_birth    | s_sex | c_id | s_score |
+------+--------+------------+-------+------+---------+
| 01   | 赵雷   | 1990-01-01 | 男    | 02   |      90 |
+------+--------+------------+-------+------+---------+
```



## 四十一 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

```mysql
select distinct a.s_id,a.c_id,a.s_score from score a,score b where a.s_id=b.s_id and a.s_score=b.s_score and a.c_id!=b.c_id;

+------+------+---------+
| s_id | c_id | s_score |
+------+------+---------+
| 03   | 01   |      80 |
| 03   | 02   |      80 |
| 03   | 03   |      80 |
+------+------+---------+
```

- `from A, B where A.id=B.id` 等价于 `from A inner join B on A.id=B.id`



## 四十二 查询每门课程成绩最好的前两名

```mysql
select
	c_id,s_id,s_score
from (
  select
    a.*,
    if (@precid!=c_id,@rownum:=1,@rownum:=@rownum+1) as row_num,
  	(case 
  		when @rownum=1 then @rank:=1
     	when @prescore=s_score then @rank
     	else @rank:=@rownum
  	end) as rank,
  	(case
     	when @rownum=1 then @denserank:=1
     	when @prescore=s_score then @denserank
      else @denserank:=@denserank+1
     end) as dense_rank,
    @precid:=c_id,
  	@prescore:=s_score
  from 
    (select * from score order by c_id,s_score desc) a,
  	(select @precid:=null,@rownum:=0,@prescore:=null,@rank:=0,@denserank:=0) b
) a where row_num in (1,2);

+------+------+---------+
| c_id | s_id | s_score |
+------+------+---------+
| 01   | 01   |      80 |
| 01   | 03   |      80 |
| 02   | 01   |      90 |
| 02   | 07   |      89 |
| 03   | 01   |      99 |
| 03   | 07   |      98 |
+------+------+---------+
```



## 四十三 统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```mysql
select c_id,count(c_id) as number from score group by c_id having number>5 order by number desc,c_id asc;

+------+--------+
| c_id | number |
+------+--------+
| 01   |      6 |
| 02   |      6 |
| 03   |      6 |
+------+--------+
```



## 四十四 检索至少选修两门课程的学生学号

```mysql
select s_id from score group by s_id having count(*)>=2;

+------+
| s_id |
+------+
| 01   |
| 02   |
| 03   |
| 04   |
| 05   |
| 06   |
| 07   |
+------+
```



## 四十五 查询选修了全部课程的学生信息

```mysql
select * from student where s_id in (
	select s_id from score group by s_id having count(*)=(select count(*) from course)
);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 01   | 赵雷   | 1990-01-01 | 男    |
| 02   | 钱电   | 1990-12-21 | 男    |
| 03   | 孙风   | 1990-05-20 | 男    |
| 04   | 李云   | 1990-08-06 | 男    |
+------+--------+------------+-------+
```



## 四十六 查询各学生的年龄

```mysql
select 
	s_id,
	s_name,
	s_birth,
	YEAR(CURRENT_DATE())-
		YEAR(s_birth)-
		IF(DATE_FORMAT(CURRENT_DATE(),'%m-%d')>=DATE_FORMAT(s_birth,'%m-%d'),0,1) as age
from student;

+------+--------+------------+------+
| s_id | s_name | s_birth    | age  |
+------+--------+------------+------+
| 01   | 赵雷   | 1990-01-01 |   30 |
| 02   | 钱电   | 1990-12-21 |   29 |
| 03   | 孙风   | 1990-05-20 |   30 |
| 04   | 李云   | 1990-08-06 |   29 |
| 05   | 周梅   | 1991-12-01 |   28 |
| 06   | 吴兰   | 1992-03-01 |   28 |
| 07   | 郑竹   | 1989-07-01 |   30 |
| 08   | 王菊   | 1990-01-20 |   30 |
+------+--------+------------+------+
```

- 考虑到了本年是否已经过了生日的问题



## 四十七 查询本周过生日的学生

```mysql
# CURRENT_DATE() 2020-06-27
select * from student where DATE_FORMAT(s_birth,'%m-%d')
between
DATE_FORMAT(DATE_ADD(CURRENT_DATE(), INTERVAL ( 7 - (WEEKDAY(CURRENT_DATE())+1) + 1 - 7 ) DAY),'%m-%d')
and
DATE_FORMAT(DATE_ADD(CURRENT_DATE(), INTERVAL ( 7 - (WEEKDAY(CURRENT_DATE())+1) ) DAY),'%m-%d');

Empty set

```



## 四十八 查询下周过生日的学生

```mysql
# CURRENT_DATE() 2020-06-27
select * from student where DATE_FORMAT(s_birth,'%m-%d')
between
DATE_FORMAT(DATE_ADD(CURRENT_DATE(), INTERVAL ( 7 - (WEEKDAY(CURRENT_DATE())+1) + 1 - 7 + 7 ) DAY),'%m-%d')
and
DATE_FORMAT(DATE_ADD(CURRENT_DATE(), INTERVAL ( 7 - (WEEKDAY(CURRENT_DATE())+1) + 7 ) DAY),'%m-%d');

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 07   | 郑竹   | 1989-07-01 | 女    |
+------+--------+------------+-------+
```



## 四十九 查询本月过生日的学生

```mysql
# CURRENT_DATE() 2020-06-27
select * from student where DATE_FORMAT(s_birth,'%m')=DATE_FORMAT(CURRENT_DATE(),'%m');

Empty set
```



## 五十 查询下月过生日的学生

```mysql
# CURRENT_DATE() 2020-06-27
select * from student where MONTH(s_birth)=(MONTH(NOW())+1);

+------+--------+------------+-------+
| s_id | s_name | s_birth    | s_sex |
+------+--------+------------+-------+
| 07   | 郑竹   | 1989-07-01 | 女    |
+------+--------+------------+-------+
```

