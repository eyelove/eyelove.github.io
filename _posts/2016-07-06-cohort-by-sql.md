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
