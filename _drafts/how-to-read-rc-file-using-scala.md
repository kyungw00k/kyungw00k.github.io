---
layout: post
title: How to read rc file using scala
date: 2015-10-22 16:52:30.000000000 +09:00
type: post
categories:
- Scala
---
```scala
val inputRDD = sc.hadoopFile[LongWritable, BytesRefArrayWritable, RCFileInputFormat[LongWritable, BytesRefArrayWritable]](path)
```

(from http://stackoverflow.com/questions/30567902/how-to-read-rc-file-using-scala)
