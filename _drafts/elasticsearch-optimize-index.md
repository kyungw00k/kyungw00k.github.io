# Optimize Index

```sh
curl -XPOST 'http://<host>:9200/_all/_optimize?only_expunge_deletes=true' > /path/to/log/optimize_index.log.`date +\%Y\%m\%d` 2>&1
```