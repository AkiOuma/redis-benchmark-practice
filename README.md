# redis benchmark practice

## 使用 redis benchmark 工具, 测试 10 20 50 100 200 1k 5k 字节 value 大小，redis get set 性能。


使用shell脚本执行以下redis benchmark工具，并把对应结果记录到同脚本目录小的`./q1/redis_($size)`文件中
```sh
echo "================ Performance of 10 Bs ================" >> result_10
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 10 >> result_10
echo "================ Performance of 20 Bs ================" >> result_20
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 20 >> result_20
echo "================ Performance of 50 Bs ================" >> result_50
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 50 >> result_50
echo "================ Performance of 100 Bs ================" >> result_100
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 100 >> result_100
echo "================ Performance of 200 Bs ================" >> result_200
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 200 >> result_200
echo "================ Performance of 1000 Bs ================" >> result_1000
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 1000 >> result_1000
echo "================ Performance of 5000 Bs ================" >> result_5000
docker exec -it redis redis-benchmark -p 6379 -t set,get -d 5000 >> result_5000
```

但是从我这边的测试结果看来，10字节到5k字节的默认100000次测试的结果看来get和set的性能没什么太大的差别.....基本都是2s到2.2s之内完成....不知道是不是因为我用的是docker版本的redis....


## 写入一定量的 kv 数据, 根据数据大小 1w-50w 自己评估, 结合写入前后的 info memory 信息 , 分析上述不同 value 大小下，平均每个 key 的占用内存空间

* 使用go-redis实例化客户端
* key使用`google/uuid`生成，而value全部使用单个字符0，测试以下数量大小的数据
  * 1w
  * 5w
  * 10w
  * 20w
  * 30w
  * 40w
  * 50w
* 每次测试完直接执行`info memory`查看当前内存信息，并将结果记录到./q2的对应文件中
* 执行flushdb命令清空缓存

记录结果如下
```
# 10000
(2.69M - 855.02K - 1B * 10000) / 10000 = 194B

# 50000
(5.51M - 1.21M - 1B *  50000) / 50000 = 89B

# 100000
(9.06M - 1.21M - 1B * 100000) / 100000 = 81B

# 200000
(9.04M - 1.21M - 1B * 200000) / 200000 = 40B

# 300000
(7.79M - 1.21M - 1B * 300000) / 300000 = 21B

# 400000
(8.82M - 1.21M - 1B * 400000) / 400000 = 19B

# 500000
(9.52M - 1.21M - 1B * 500000) / 500000 = 16B
```

结论似乎是同等字符长度的key会随着数据量的增大，在内存中占据的空间越小...