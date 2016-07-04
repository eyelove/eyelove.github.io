---
layout: post
fbcomments: false
published: true
title: Redis Client 일괄 삭제
category: redis
tags:
  - cli
  - Redis
---

redis에 import하는 배치에서 connection close가 정상적으로 이뤄지지 않아서 connection이 죽지 않는 문제가 발생했다.
아래는 redis에 접속해있는 client중 특정 아이피를 찾은 후 특정작업에 대해서 일괄적으로 kill 해주는 스크립트 이다.

```
redis-cli  -p 6379  CLIENT LIST | grep "192.168.0.1" | grep "sadd\|srem" | cut -d ' ' -f 2 | cut -d = -f 2 | awk '{ print "CLIENT KILL " $0 }' | redis-cli -p 6379 -x
```
