# mysql

## case when

```sql
select name as '名字', (case sex when 0 then '女' else '男' end) as '性别'
from student;
```

```sql
-- 列转行操作
-- 先按照科目分开， 符合条件的设置分数，不符合的给置零。
select name as '姓名'
,(case course when '语文' then score else 0 end) as '语文'
,(case course when '数学' then score else 0 end) as '数学'
,(case course when '英语' then score else 0 end) as '英语'
from course_score

-- 然后再按照名字group by ，对分数求max。
select name as '姓名'
,max(case course when '语文' then score else 0 end) as '语文'
,max(case course when '数学' then score else 0 end) as '数学'
,max(case course when '英语' then ifnull(score,0) else 0 end) as '英语'
from course_score group by name;

SELECT name as '姓名'
max(IF(`subject`='语文',score,0)) as '语文',
max(IF(`subject`='数学',score,0)) as '数学',
max(IF(`subject`='英语',score,0)) as '英语',
max(IF(`subject`='政治',ifnull(score,0),0)) as '政治' 
FROM course_score 
GROUP BY name


```

https://blog.csdn.net/weter_drop/article/details/105899362?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-3&spm=1001.2101.3001.4242



1. 通过group_concat函数将value列的值拼接成一个逗号隔开的字符串，然后通过substring_index函数对字符串进行截取
2. 通过substring_index函数特性，我们就需要知道字符串有多少个逗号，并且要告诉每个逗号的位置
3. 逗号个数=char_length(字符串)-char_length(replace(字符串,',',''))
4. 逗号位置=mysql.help_topic.id < 逗号个数[+1]
5. 最后通过distinct函数将截取后的单个值进行去重