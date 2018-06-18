---
title: 2018-6-18-Migrate-Sentry
date: 2018-06-18 10:36:58
tags: [Nginx]
categories: Nginx
---

Sentry 在高并发期还是很占资源的。所以将其迁移出业务服务器。

<!-- more -->

---

### 清理数据

```
sentry cleanup -days 30 -q
vacuumdb -U postgres -d 0532Shanjing -t nodestore_node -v -f --analyze
```

### 备份数据
```
//方法1
pg_dump  -U postgres 0532Shanjing -f /backup/postgres_0532Shanjing_2018-06-16_1.dump

//方法2
sentry --config /etc/sentry export  --exclude
```