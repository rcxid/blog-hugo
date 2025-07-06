+++
title = "SQL统计连续登录用户问题"
date = "2025-07-06T12:11:26+08:00"
summary = "SQL统计连续登录用户问题：LEAD和ROW_NUMBER函数实现"
categories = ["数据处理"]
tags = ["hive", "sql"]
draft = false
+++

## 问题描述

统计一个月中，存在7天连续登录的用户数量。

## 数据示例：使用LEAD函数实现
| user_id | login_date | date_add_6_day（sql新增列） | date_next_6_row（sql新增列） |
| ------- | ---------- | ---------------------- | ----------------------- |
| 0001    | 2020-01-02 | 2020-01-08             | 2020-01-10              |
| 0001    | 2020-01-03 | 2020-01-09             | 2020-01-11              |
| 0001    | 2020-01-04 | 2020-01-10             | 2020-01-12              |
| 0001    | 2020-01-07 | **2020-01-13**         | **2020-01-13**          |
| 0001    | 2020-01-08 | 2020-01-14             | 9999-12-31              |
| 0001    | 2020-01-09 | 2020-01-15             | 9999-12-31              |
| 0001    | 2020-01-10 | 2020-01-16             | 9999-12-31              |
| 0001    | 2020-01-11 | 2020-01-17             | 9999-12-31              |
| 0001    | 2020-01-12 | 2020-01-18             | 9999-12-31              |
| 0001    | 2020-01-13 | 2020-01-19             | 9999-12-31              |
### 计算SQL

```sql
-- 添加字段，标注存在连续7天登录的用户
with t1 as (
  select
    user_id,
    case
      when date_add(login_date, 6) = lead(login_date, 6, '9999-12-31') over(
        partition by user_id
        order by login_date asc
      ) then 1
      else 0
    end as login_flag
  from table_name
)

select
  count(*) as user_count
from (
  select
    user_id
  from t1
  where login_flag = 1
  group by user_id
) t;
```

### 数据示例：使用ROW_NUMBER函数实现
| user_id | login_date | rn（row_number） | date_sub_rn（sql新增列） |
| ------- | ---------- | -------------- | ------------------- |
| 0001    | 2020-01-02 | 1              | 2020-01-01          |
| 0001    | 2020-01-03 | 2              | 2020-01-01          |
| 0001    | 2020-01-04 | 3              | 2020-01-01          |
| 0001    | 2020-01-07 | 4              | 2020-01-03          |
| 0001    | 2020-01-08 | 5              | 2020-01-03          |
| 0001    | 2020-01-09 | 6              | 2020-01-03          |
| 0001    | 2020-01-10 | 7              | 2020-01-03          |
| 0001    | 2020-01-11 | 8              | 2020-01-03          |
| 0001    | 2020-01-12 | 9              | 2020-01-03          |
| 0001    | 2020-01-13 | 10             | 2020-01-03          |
### 计算SQL

```sql
-- 添加字段
with t1 as (
  select
    user_id,
    date_sub(login_date,
      row_number() over (
        partition by user_id
        order by login_date asc
      )
    ) as date_sub_rn
  from table_name
)

select
  count(distinct user_id) as user_count
from (
  select
    user_id, date_sub_rn, count(*) as login_cnt
  from t1
  group by user_id, date_sub_rn
) t
where login_cnt >= 7;
```