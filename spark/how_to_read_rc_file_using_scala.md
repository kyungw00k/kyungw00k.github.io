# How to read rc file using scala

```
val inputRDD = sc.hadoopFile[LongWritable, BytesRefArrayWritable, RCFileInputFormat[LongWritable, BytesRefArrayWritable]](path)
```

(from http://stackoverflow.com/questions/30567902/how-to-read-rc-file-using-scala)