---
layout: post
fbcomments: false
published: true
title: Cohort by sql
tags:
  - db
---
Cohort 차트를 생성할 시 필요한 테이타를 추출하는 SQL예제이다.

> with문법은 때론 동작하지 않으므로 별도 테이블로 생성하는 것도 방법이다. DB특성에 따라 변경한다.

```sql
with

users as (
  select
  	userid,
  	min(to_date(created, 'YYYY-MM-DD')) as activated_at
  from logtable
  group by 1
),

events as (
  select
  	userid,
  	to_date(created, 'YYYY-MM-DD') as occurred_at
  from logtable
)


select
x.activated_at,
sum(case when x.user_period = 0 then retained_users end) as d0,
sum(case when x.user_period = 1 then retained_users end) as d1,
sum(case when x.user_period = 2 then retained_users end) as d2,
sum(case when x.user_period = 3 then retained_users end) as d3,
sum(case when x.user_period = 4 then retained_users end) as d4,
sum(case when x.user_period = 5 then retained_users end) as d5,
sum(case when x.user_period = 6 then retained_users end) as d6,
sum(case when x.user_period = 7 then retained_users end) as d7,
sum(case when x.user_period = 8 then retained_users end) as d8,
sum(case when x.user_period = 9 then retained_users end) as d9,
sum(case when x.user_period = 10 then retained_users end) as d10
from (
select
	u.activated_at,
	datediff(day, u.activated_at, e.occurred_at) as user_period,
	count(DISTINCT e.userid) as retained_users
from users u
join events e on e.userid = u.userid and e.occurred_at >= u.activated_at
group by 1,2 ) x
group by 1
order by 1;
```

- mysql : datediff(종료일, 시작일)
- redshift : datediff(datepart[day, week, ...], 시작일, 종료일)

```
 activated_at |  d0   |  d1   |  d2   |  d3   |  d4   |  d5   |  d6   |  d7   |  d8   |  d9   | d10
--------------+-------+-------+-------+-------+-------+-------+-------+-------+-------+-------+-----
 2016-05-11   | 74468 | 50448 | 54202 | 56750 | 52483 | 53334 | 48252 | 50954 | 47418 | 52329 |
 2016-05-12   | 11034 |  4628 |  5089 |  4371 |  4159 |  3708 |  3948 |  3715 |  4441 |       |
 2016-05-13   |  7978 |  3709 |  2988 |  2714 |  2281 |  2434 |  2279 |  2815 |       |       |
 2016-05-14   |  8229 |  2964 |  2192 |  1811 |  1941 |  1778 |  2156 |       |       |       |
 2016-05-15   |  4931 |  1449 |  1183 |  1203 |  1064 |  1280 |       |       |       |       |
 2016-05-16   |  3358 |  1097 |  1047 |   940 |  1033 |       |       |       |       |       |
 2016-05-17   |  3091 |  1161 |  1019 |  1062 |       |       |       |       |       |       |
 2016-05-18   |  3203 |  1150 |  1191 |       |       |       |       |       |       |       |
 2016-05-19   |  3778 |  1607 |       |       |       |       |       |       |       |       |
 2016-05-20   |  8420 |       |       |       |       |       |       |       |       |       |
```