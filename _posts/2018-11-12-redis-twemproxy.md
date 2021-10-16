---
layout: post
title: redis
category: redis
tags: [redis]
---

# Redis

### redis info

```text
>info memory
used_memory:3919432
used_memory_human:3.74M
used_memory_rss:13938688
used_memory_rss_human:13.29M
used_memory_peak:4553248
used_memory_peak_human:4.34M
total_system_memory:10481418240
total_system_memory_human:9.76G
used_memory_lua:41984
used_memory_lua_human:41.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:3.56
mem_allocator:jemalloc-4.0.3
```

**used\_memory**：Redis分配器分配的内存总量（单位是字节），包括使用的虚拟内存（即swap\); used\_memory\_human只是显示更友好。

**used\_memory\_rss**:  Redis进程占据操作系统的内存（单位是字节），与top命令的RES\(Resident Memory\)常驻内存字段一致；除了分配器分配的内存之外，used\_memory\_rss还包括进程运行本身需要的内存、内存碎片等，_但是不包括虚拟内存_。

> used\_memory和used\_memory\_rss，前者是从Redis角度得到的量，后者是从操作系统角度得到的量。二者之所以有所不同，一方面是因为内存碎片和Redis进程运行需要占用内存，使得前者可能比后者小，另一方面虚拟内存的存在，使得前者可能比后者大

> 由于在实际应用中，Redis的数据量会比较大，此时进程运行占用的内存与Redis数据量和内存碎片相比，都会小得多；因此used\_memory\_rss和used\_memory的比例，便成了衡量Redis内存碎片率的参数；这个参数就是mem\_fragmentation\_ratio。

**mem\_fragmentation\_ratio:** 内存碎片比率，该值是used\_memory\_rss / used\_memory的比值。
- `mem_fragmentation_ratio`一般大于1，且该值越大，内存碎片比例越大。mem\_fragmentation\_ratio&lt;1，说明Redis使用了虚拟内存，由于虚拟内存的媒介是磁盘，比内存速度要慢很多，当这种情况出现时，应该及时排查，如果内存不足应该及时处理，如增加Redis节点、增加Redis服务器的内存、优化应用等。
-  一般来说，`mem_fragmentation_ratio`在1.03左右是比较健康的状态（对于jemalloc来说）；上面截图中的`mem_fragmentation_ratio`值很大，是因为还没有向Redis中存入数据，Redis进程本身运行的内存使得`used_memory_rss` 比`used_memory大`得多。

# redis twemproxy集群架构

由 Twitter 开源的 Twemproxy 具有如下优点：

* 性能很好且足够稳定，自建内存池实现 Buffer 复用，代码质量很高；
* 支持 `fnv1a_64`、`murmur`、`md5` 等多种哈希算法；
* 支持一致性哈希（ketama），取模哈希（modula）和随机（random）三种分布式算法。

但是缺点也很明显：

* 单核模型造成性能瓶颈；
* 传统扩容模式仅支持停机扩容。

![](/images/posts/redis/redis-cluster.png)

# redis 监控工具


1. redis-stat  
   * docker run --name redis-stat -p 8080:63790 -d insready/redis-stat --server 192.168.99.100
2. redis-live
   * docker run --name redis-live -p 8888:8888 -d snakeliwei/redislive
3. remdom
   * docker run -p 4567:4567 -d vieux/redmon -r redis://192.168.99.100:6379
4. Redis Commander
   * docker run --rm --name redis-commander -d  --env REDIS\_HOSTS=10.10.20.30  -p 8081:8081  rediscommander/redis-commander:latest

redis 内存分析工具 

1. Redis-audit
   * [https://github.com/snmaynard/redis-audit](https://github.com/snmaynard/redis-audit)
2. redis-samper
   * [https://github.com/antirez/redis-sampler](https://github.com/antirez/redis-sampler)
3. Redis Toolkit
   * [https://github.com/alexdicianu/redis\_toolkit](https://github.com/alexdicianu/redis_toolkit)
4. redis-memory-analyzer
   * [https://github.com/gamenet/redis-memory-analyzer](https://github.com/gamenet/redis-memory-analyzer)
5. redis-faina
   * [https://github.com/facebookarchive/redis-faina](https://github.com/facebookarchive/redis-faina)

比较推荐 redis-live + redis-memory-analyzer + redis-faina

### redis-memory-analyzer\(rma\) 使用教程

#### Introduction

RMA是一个控制台工具，可实时扫描Redis key space并按key模式汇总内存使用情况统计信息。可以按所有或选定的Redis类型进行扫描，例如“字符串”，“哈希”，“列表”，“设置”，“ zset”，并根据需要使用匹配模式。RMA尝试通过模式来区分键名称，例如，如果具有诸如'user:100' and 'user:101''之类的键，应用程序会在输出中选择常见的模式'user:\*'，你可以分析自己内存中的大多数内存不良数据实例。

在大型数据库情况下尝试运行使用--limit选项限制key的数量，--types 指定key类型。有三种模式可以选择global', 'scanner', 'ram' and 'all'.

* global: 显示系统信息和redisKeySpaceOverhead
* scanner:显示哪种key存储在redis,哪种数据结构最常使用在系统
* ram: 数据相关输出 输出结果以key value模式显示。（单位byte）
* all: 默认模式 输出所有信息包含（global scanner ram）

#### Geting Start

string类型   
set "name" "wenxue"

```text
rma -s 10.10.5.108 -p 7800 -m name

Keys by types

| name   |   count | type   | percent   |
|:-------|--------:|:-------|:----------|
| name   |	 1 | string | 0.40%     |

Keys statistic

Stat by <string>

| Match   |   Count |   Useful |   Free |   Real |   Ratio | Encoding        |   Min |   Max |   Avg |   TTL Min |   TTL Max |   TTL Avg |
|:--------|--------:|---------:|-------:|-------:|--------:|:----------------|------:|------:|------:|----------:|----------:|----------:|
| name    |	  1 |        6 |      0 |     48 |    8.00 | embstr [100.0%] |     6 |     6 |     6 |        -1 |        -1 |     -1.00 |
| Total:  |	  1 |        6 |      0 |     48 |    0.00 |                 |     0 |     0 |     0 |        -1 |        -1 |    nan    |

```

 wenxue 6个字节 对应userful min max字段，Real字段代表了实际分配的内存，在redis中以raw \(sds string\)编码的key,实际分配内存公式为 align\(redis object\) + align\(sds header + useful payload\)=33+\(9+6\)=48  
ratio:期望内存和实际内存的·比例 即48/6=8.0  
free: 表示没有使用但分配的内存被raw编码的sds string。

hash 类型  
HMSET runoobkey name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000

```text
Keys by types

| name      |   count | type   | percent   |
|:----------|--------:|:-------|:----------|
| runoobkey |       1 | hash   | 0.38%     |

Keys statistic

Stat by <hash>

| Match     |   Count |   Avg field count |   Key mem |   Real |   Ratio |   Value mem |   Real |   Ratio |   System | Encoding         |   Total mem |   Total aligned |   TTL Min |   TTL Max |   TTL Avg. |
|:----------|--------:|------------------:|----------:|-------:|--------:|------------:|-------:|--------:|---------:|:-----------------|------------:|----------------:|----------:|----------:|-----------:|
| runoobkey |       1 |                 4 |        28 |     44 |    1.57 |          53 |     64 |    1.21 |	  96 | ziplist [100.0%] |          81 |             204 |        -1 |        -1 |      -1.00 |
| Total:    |       1 |                 0 |        28 |     44 |    0.00 |          53 |     64 |    0.00 |	  96 |                  |          81 |             204 |        -1 |        -1 |     nan    |
```

Avg field count： runoobkey对象有4个filed   
key mem: name descripttion likes visitors 占用的字节数   
value mem: 对应字段value占用的字节

list类型  
LPUSH alist redis  
LPUSH alist mysql

```text
Keys by types

| name   |   count | type   | percent   |
|:-------|--------:|:-------|:----------|
| alist  |	 1 | list   | 0.37%     |

Keys statistic

Stat by <list>

| Match   |   Count |   Avg Count |   Min Count |   Max Count |   Stdev Count |   Value mem |   Real |   Ratio |   System | Encoding           |   Total |   TTL Min |   TTL Max |   TTL Avg |
|:--------|--------:|------------:|------------:|------------:|--------------:|------------:|-------:|--------:|---------:|:-------------------|--------:|----------:|----------:|----------:|
| alist   |	  1 |           2 |           2 |           2 |             0 |          10 |     32 |    3.20 |       64 | quicklist [100.0%] |      96 |        -1 |        -1 |     -1.00 |
| Total:  |	  1 |           0 |           0 |           0 |             0 |          10 |     32 |    0.00 |       64 |                    |      96 |        -1 |        -1 |    nan    |

```

set类型  
SADD aset redis   
SADD aset mysql

```text
Keys by types

| name   |   count | type   | percent   |
|:-------|--------:|:-------|:----------|
| aset   |	 1 | set    | 0.37%     |

Keys statistic

Stat by <set>

| Match   |   Count |   Avg Count |   Value mem |   Real |   Ratio |   System* | Encoding           |   Total |   TTL Min |   TTL Max |   TTL Avg. |
|:--------|--------:|------------:|------------:|-------:|--------:|----------:|:-------------------|--------:|----------:|----------:|-----------:|
| aset    |	  1 |           2 |          10 |     96 |    9.60 |	   128 | hashtable [100.0%] |     224 |        -1 |        -1 |      -1.00 |
| Total:  |	  1 |           0 |          10 |     96 |    0.00 |	   128 |                    |     224 |        -1 |        -1 |     nan    |


```

zset类型  
ZADD zzset 1 redis   
ZADD zzset 3 mysql

```text
Keys by types

| name   |   count | type   | percent   |
|:-------|--------:|:-------|:----------|
| zzset  |	 1 | zset   | 4.54%     |

Keys statistic

Stat by <zset>

| Match   |   Count |   Useful |   Real |   Ratio | Encoding        |   Min |   Max |   Avg |   TTL Min |   TTL Max |   TTL Avg. |
|:--------|--------:|---------:|-------:|--------:|:----------------|------:|------:|------:|----------:|----------:|-----------:|
| zzset   |	  1 |        5 |     48 |    9.60 | embstr [100.0%] |     5 |     5 |     5 |        -1 |        -1 |      -1.00 |
| Total:  |	  1 |        5 |     48 |    0.00 |                 |     0 |     0 |     0 |        -1 |        -1 |     nan    |
```



     
