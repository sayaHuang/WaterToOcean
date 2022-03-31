# spark 小文件问题

## 为什么会存在小文件问题

## 解决办法
### 1.合并小文件-combine
**使用场景 **当SQL逻辑中包含Shuffle操作时。

**原因说明: **Spark SQL的表中，经常会存在很多小文件（大小远小于HDFS块大小），每个小文件默认对应Spark中的一个Partition，也就是一个Task。在很多小文件场景下，Spark会起很多Task。当SQL逻辑中存在Shuffle操作时，会大大增加hash分桶数，严重影响性能。

**配置参数: **在小文件场景下，您可以通过如下配置手动指定每个Task的数据量（Split Size），确保不会产生过多的Task，提高性能。

![combine](./image/Snipaste_2022-03-31_21-30-55.png)
