---
title: Day48-数据库（五）
date: 2022-07-19 10:53:22
categories:
- Python
- 03_MySQL
tags:
---

### 知识点补充

```python
# 查询平均年龄在25岁以上的部门名称
"""只要是多表查询 就有两种思路    联表    子查询"""
# 联表操作
	1 先拿到部门和员工表 拼接之后的结果
	2 分析语义 得出需要进行分组
    select dep.name from emp inner join dep
    	on emp.dep_id = dep.id
        group by dep.name
        having avg(age) > 25
        ;
	"""涉及到多表操作的时候 一定要加上表的前缀"""
# 子查询
	select name from dep where id in
		(select dep_id from emp group by dep_id 
    		having avg(age) > 25);

# 关键字exists(了解)
	只返回布尔值 True False
    返回True的时候外层查询语句执行
    返回False的时候外层查询语句不再执行
	select * from emp where exists 
    	(select id from dep where id>3);
        
        
   select * from emp where exists 
    	(select id from dep where id>300);
```



# 今日内容概要

* navicat可视化界面操作数据库
* 数据库查询题目讲解(多表操作)
* python如何操作MySQL(pymysql模块)
* sql注入问题
* pymysql模块增删改查数据操作

### 一、Navicat软件

```python
"""
一开始学习python的时候 下载python解释器然后直接在终端书写
pycharm能够更加方便快捷的帮助你书写python代码
excel word pdf

我们在终端操作MySQL 也没有自动提示也无法保存等等 不方便开发
Navicat内部封装了所有的操作数据库的命令 
用户在使用它的时候只需要鼠标点点即可完成操作 无需书写sql语句
"""
```

**安装**

```python
直接百度搜索 有破解版的也有非破解
非破解的有试用期 你如果不嫌麻烦 你就用使用
到期之后重新装再使用 或者破解一下也很简单
https://www.cr173.com/soft/126934.html
    
下载完成后是一个压缩包 直接解压 然后点击安装 有提醒直接点击next即可

navicat能够充当多个数据库的客户端


navicat图形化界面有时候反应速度较慢 你可以选择刷新或者关闭当前窗口再次打开即可

当你有一些需求该软件无法满足的时候 你就自己动手写sql

```

### 提示

```python
"""
1 MySQL是不区分大小写的
	验证码忽略大小写
		内部统一转大写或者小写比较即可
			upper
			lower

2 MySQL建议所有的关键字写大写

3 MySQL中的注释 有两种
	--
	#

4 在navicat中如何快速的注释和解注释
	ctrl + ？  加注释
	ctrl + ？  基于上述操作再来一次就是解开注释
	如果你的navicat版本不一致还有可能是
	ctrl + shift + ？解开注释
"""
```

### 练习题

```python
"""
课下一定要把握上课将的这几道题全部自己独立的理解并写出来

在解决sql查询问题的时候 不要慌
一步一步慢慢来  最终能够东拼西凑出来就过关了！！！

"""
-- 1、查询所有的课程的名称以及对应的任课老师姓名
-- SELECT
-- 	course.cname,
-- 	teacher.tname 
-- FROM
-- 	course
-- 	INNER JOIN teacher ON course.teacher_id = teacher.tid;

-- 4、查询平均成绩大于八十分的同学的姓名和平均成绩
-- SELECT
-- 	student.sname,
-- 	t1.avg_num 
-- FROM
-- 	student
-- 	INNER JOIN (
-- 	SELECT
-- 		score.student_id,
-- 		avg( num ) AS avg_num 
-- 	FROM
-- 		score
-- 		INNER JOIN student ON score.student_id = student.sid 
-- 	GROUP BY
-- 		score.student_id 
-- 	HAVING
-- 		AVG( num ) > 80 
-- 	) AS t1 ON student.sid = t1.student_id;


-- 7、 查询没有报李平老师课的学生姓名
# 分步操作
# 1 先找到李平老师教授的课程id
# 2 再找所有报了李平老师课程的学生id
# 3 之后去学生表里面取反 就可以获取到没有报李平老师课程的学生姓名
-- SELECT
-- 	student.sname 
-- FROM
-- 	student 
-- WHERE
-- 	sid NOT IN (
-- 	SELECT DISTINCT
-- 		score.student_id 
-- 	FROM
-- 		score 
-- 	WHERE
-- 		score.course_id IN ( SELECT course.cid FROM teacher INNER JOIN course ON teacher.tid = course.teacher_id WHERE teacher.tname = '李平老师' ) 
-- 	);

-- 8、 查询没有同时选修物理课程和体育课程的学生姓名
--     (只要选了一门的 选了两门和没有选的都不要)
# 1 先查物理和体育课程的id
# 2 再去获取所有选了物理和体育的学生数据
# 3 按照学生分组 利用聚合函数count筛选出只选了一门的学生id
# 4 依旧id获取学生姓名
-- SELECT
-- 	student.sname 
-- FROM
-- 	student 
-- WHERE
-- 	student.sid IN (
-- 	SELECT
-- 		score.student_id 
-- 	FROM
-- 		score 
-- 	WHERE
-- 		score.course_id IN ( SELECT course.cid FROM course WHERE course.cname IN ( '物理', '体育' ) ) 
-- 	GROUP BY
-- 		score.student_id 
-- 	HAVING
-- 		COUNT( score.course_id ) = 1 
-- 	);

-- 9、 查询挂科超过两门(包括两门)的学生姓名和班级
# 1 先筛选出所有分数小于60的数据
# 2 按照学生分组 对数据进行计数获取大于等于2的数据
SELECT
	class.caption,
	student.sname 
FROM
	class
	INNER JOIN student ON class.cid = student.class_id 
WHERE
	student.sid IN (
	SELECT
		score.student_id 
	FROM
		score 
	WHERE
		score.num < 60 GROUP BY score.student_id HAVING COUNT( score.course_id ) >= 2 
	);
```

### pymysql模块

```python
"""
支持python代码操作数据库MySQL
"""
pip3 install pymysql
```

### sql注入

```python
"""
利用一些语法的特性 书写一些特点的语句实现固定的语法
MySQL利用的是MySQL的注释语法
select * from user where name='jason' -- jhsadklsajdkla' and password=''

select * from user where name='xxx' or 1=1 -- sakjdkljakldjasl' and password=''
"""
日常生活中很多软件在注册的时候都不能含有特殊符号
因为怕你构造出特定的语句入侵数据库 不安全

# 敏感的数据不要自己做拼接 交给execute帮你拼接即可
# 结合数据库完成一个用户的登录功能？
import pymysql


conn = pymysql.connect(
    host = '127.0.0.1',
    port = 3306,
    user = 'root',
    password = '123456',
    database = 'day48',
    charset = 'utf8'  # 编码千万不要加-
)  # 链接数据库
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)

username = input('>>>:')
password = input('>>>:')
sql = "select * from user where name=%s and password=%s"
# 不要手动拼接数据 先用%s占位 之后将需要拼接的数据直接交给execute方法即可
print(sql)
rows = cursor.execute(sql,(username,password))  # 自动识别sql里面的%s用后面元组里面的数据替换
if rows:
    print('登录成功')
    print(cursor.fetchall())
else:
    print('用户名密码错误')
```

**作业布置**

```python
"""
1 navicat自己玩一玩
2 练习题一定要搞懂 照着我的思路一遍遍的看敲
3 熟悉pymysql的使用
4 sql注入产生的原因和解决方法 了解
5 思考:如何结合mysql实现用户的注册和登录功能？
"""
```































