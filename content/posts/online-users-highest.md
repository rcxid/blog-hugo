+++
title = "SQL统计在线人数达到最大值的时刻"
date = "2025-07-05T23:37:22+08:00"
summary = "SQL统计在线人数达到最大值的时刻"
categories = ["数据分析"]
tags = ["hive", "sql"]
draft = false
+++

## 问题描述

有一张用户行为日志表（user_action_log），记录了用户登录和下线的操作日志数据。有三个关键字段：user_id，event_id（001为登录日志，002为下线日志），event_time。表中的数据不保证按照时间有序排列，还可能存在脏数据，例如，一个用户没有登录，却存在下线日志（跨天）；登录了，后续出现连续多次下线日志等。正常的数据顺序是，先出现登录日志，再出现下线日志，交替出现，最后可以有下线日志，也可以没有。现在给定某天的数据，计算那一天在那个时间，系统的在线人数达到了最大值。

### 数据示例：

| user_id | event_id | event_time    | day        |
| ------- | -------- | ------------- | ---------- |
| 1       | 001      | 1601960734434 | 2020-10-06 |
| 2       | 001      | 1601960735643 | 2020-10-06 |
| 3       | 001      | 1601960739765 | 2020-10-06 |
| 2       | 002      | 1601960786356 | 2020-10-06 |
| 1       | 002      | 1601960877346 | 2020-10-06 |

## 计算SQL

```sql
-- 一个用户的login和logout时间相同时，排序时，login数据需要排在logout前面
-- 脏数据标记过滤：确保用户的login、logout数据交叉出现
with source_data as (
  select
    user_id, event_id, event_time,
    -- 脏数据标记字段
    case
	  -- 如果第一条是login保留
	  when event_id = '001' and row_number() over (
	    partition by user_id 
	    order by event_time asc, event_id asc
	  ) = 1 then 1
	  -- 如果当前一条和前一条不一样，保留
	  when event_id != lag(event_id, 1, 'null') over (
	    partition by user_id 
	    order by event_time asc, event_id asc
	  ) then 1
	  -- 脏数据标记
	  else 0
    end as remain_flag
  from user_action_log
  where day = '2020-10-06'
    and event_id in ('001', '002')
),

-- 过滤掉脏数据，开窗生成统计聚合字段
stat_data as (
  select
    user_id, event_time,
    sum(if(event_id = '001', 1, -1)) over (
      order by event_time asc, event_id asc
      rows between unbounded preceding and current row
    ) as current_online_count
  from source_data
  where remain_flag = 1
)

-- 选出在线最大时的那一条数据，包含登录的用户和登录时间，那个时间即为最高在线人数的时刻
select
  user_id, event_time, current_online_count
from (
  select
    user_id, event_time,
    current_online_count,
    row_number() over (order by current_online_count desc) as rn
  from stat_data
) t
where rn = 1;
```