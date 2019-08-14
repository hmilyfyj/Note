---
title: Chunk、ChunkById
date: 2019-06-30 18:38
tags: 折腾,Docker
categories: Docker
---

最近为了处理批量数据库查询，一开始使用了 chunk 方法。

<!-- more -->

---

先说结论：最好使用`ChunkById`

踩坑：使用`ChunkById`时不要用 OrderBy，否则当数据超过分页时，会导致数据重复。