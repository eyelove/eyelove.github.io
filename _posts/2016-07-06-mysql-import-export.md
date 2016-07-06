---
layout: post
fbcomments: false
published: true
title: 'mysql import / export '
tags:
  - db
---
MySql에서 csv파일로 import/export하는 쿼리문을 기록한다.

#### mysql
```sql
// import
LOAD DATA LOCAL INFILE "/tmp/filename.csv" 
INTO TABLE [table name] FIELDS TERMINATED BY ","
IGNORE 1 LINES
(col1,col2,@dummy,col4,col3);

// export
select col1 from table
INTO OUTFILE '/tmp/filename.csv'
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
ESCAPED BY '\\'
LINES TERMINATED BY '\n';
```
- 파일을 읽어올 때는 csv파일기준으로 컬럼을 매핑하며, skip이 필요할 경우 '@dummy'를 사용한다.


#### Redshift
```sql
// import
copy [table name] (col1, col2, col3)
from 's3://bucket/filename.csv'
credentials 'aws_access_key_id=######;aws_secret_access_key=########'
format csv
IGNOREHEADER 1;

// export
unload ('select * from table limit 10') 
to 's3://mybucket/venue_pipe_' 
credentials 'aws_access_key_id=######;aws_secret_access_key=########'; 
```
