---
title: 'Redis基本命令'
author: ['kyle']
description: "来源于多年前记录的笔记"
date: '2025-07-26T16:25:28+08:00'
tags:
- Redis

keywords:
- Redis
---

## set key value 添加/修改值

```sh
set key1 test-text
```

## string

### get key 获取值

```sh
get key1
```

### del key 删除值

```sh
del key1
```

### setex key time value 设置值并设置超时时间，超时自动清除

```sh
# 10秒后清除
setex key1 10 test-text
```

### mset key value [key2 value2 ...] 批量添加/修改值

```sh
mset key1 value1 key2 value2
```

### mget key [key2 ...] 批量获取值

```sh
mget key1 key2

# 返回：
# 1> xxx
# 2> yyy
```

## hash

### hset key field value 设置值

```sh
hset key1 name bob
hset key1 tag coder
```

### hget key field 获取值

```sh
hget key1 name
hget key1 tag
```

### hgetall key 获取值（全部）

```sh
hgetall key1

# 返回：
# 1> name
# 2> bob
# 3> tag
# 4> coder
```

### hmset key field value [field2 value2 ...] 设置值(多字段)

```sh
hmset key1 name bob tag coder
```

### hmget key field field2 获取值(多字段)

```sh
hmget key1 name tag

# 返回：
# 1> bob
# 2> coder
```

### hlen key 获取该hash有多少个字段

```sh
hlen key1

# 返回：
# <integer> 2
```

### hexists key field 判断该hash是否有该字段

```sh
hexists key1 name
```

## list

### lpush/rpush key value1 [value2 ...] 添加值到key list

> lpush添加到链表头部（左边）
> 
> rpush添加到链表尾部（右边）

```sh
lpush key1 i1 i2 i3
# 1> i3
# 2> i2
# 3> i1
rpush key1 i1 i2 i3
# 1> i1
# 2> i2
# 3> i3
```

### lrange key start end 获取指定区间内元素

> start和end表示开始（包含）和结束（包含）
> 
> start和end负数表示倒数第n个，0左边第一位，-1表示最后一位

```sh
lrange key1 0 -1
```

### lpop/rpop key 获取并弹出元素

> lpop弹出链表头部元素（左边）
> 
> rpop弹出链表尾部元素（右边）

```sh
lpop key1
rpop key1
```

### lindex key index 获取指定位置元素

```sh
lindex key1 1
```

### llen key 获取list长度

## set

### sadd key value1 [value2 ...] 添加元素到set

```sh
sadd key1 b a c

# 1> a
# 2> b
# 3> c
```

### smembers key 取出全部set元素

```sh
smembers key1

# 1> a
# 2> b
# 3> c
```

### sismember key value 判断set是否包含元素

```sh
sismember key1 b
```

### srem key value 删除指定set元素

```sh
srem key1 b
```