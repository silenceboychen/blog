---
title: MySQL查询表内重复记录
toc: true
date: 2016-08-04 10:02:52
categories:
tags:
---

1、查找表中多余的重复记录，重复记录是根据单个字段（user_id）来判断

```
select * from tbl_user where user_id in (select user_id from people group by user_id having count(user_id) > 1)
```

2、删除表中多余的重复记录，重复记录是根据单个字段（user_id）来判断，只留有一个记录

```
delete from tbl_user where user_id in (select user_id from people group by user_id having count(user_id) > 1) and min(id) not in (select id from people group by user_id having count(user_id)>1)
```
3、查找表中多余的重复记录（多个字段）

```
select * from table where (user_id,lesson_id) in (select user_id,lesson_id from table group by user_id,lesson_id having count(*) > 1)
```

4、删除表中多余的重复记录（多个字段），只留有id最小的记录

```
delete from table where (user_id,lesson_id) in (select user_id,lesson_id from table group by user_id,lesson_id having count(*) > 1) and id not in (select min(id) from table group by user_id,lesson_id having count(*)>1)
```
5、查找表中多余的重复记录（多个字段），不包含id最小的记录

```
select * from table where (user_id,lesson_id) in (select user_id,lesson_id from table group by user_id,lesson_id having count(*) > 1) and id not in (select min(id) from table group by user_id,lesson_id having count(*)>1)
```