---
layout: post
title: Optimize Index
date: 2016-01-20 11:07:44.000000000 +09:00
type: post
categories:
- ElasticSearch
---
```sh
curl -XPOST 'http://<host>:9200/_all/_optimize?only_expunge_deletes=true' > /path/to/log/optimize_index.log.`date +\%Y\%m\%d` 2>&1
```
