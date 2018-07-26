---
layout: post
title: mysql操作json类型(关于路径或键值对)
key: szk
tags: MySql
---

## 查找json列不包含某个键的行

假设有一数据表名为**resource_obj**,其中列**json_attrs**为JSON类型。
现在有这样一个需求：
- 当**aclNumber**不为空时，返回的是**json_attrs**列中含有*acl_number*键且值为**'aclNumber'**的行
- 当**aclNumber**为空时，返回的是**json_attrs**列中**不含有***acl_number*键的行

可以用choose来做判断，我们主要看一下otherwise中的实现。

``` mysql
<choose>
	<when test="aclNumber != null and aclNumber != ''">
		and JSON_EXTRACT(resource_obj.json_attrs,'$.acl_number') = #{aclNumber,jdbcType=VARCHAR}
	</when>
	<otherwise>
		and JSON_CONTAINS_PATH(resource_obj.json_attrs,'one','$.acl_number') = '0'
	</otherwise>
</choose>

```
<!--more-->
## JSON_CONTAINS_PATH
**JSON_CONTAINS_PATH**用于判断一个json串中是否存在某一路径。存在则返回1，不存在返回0。
其查询的是路径或key是否存在，不涉及值。
JSON_CONTAINS(column,'one/all',paths)

- all指代路径都存在时返回1，
- one指代路径只要存在一个就返回1

```
mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e');
+---------------------------------------------+
| JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e') |
+---------------------------------------------+
|                                           1 |
+---------------------------------------------+
mysql> SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e');
+---------------------------------------------+
| JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e') |
+---------------------------------------------+
|                                           0 |
+---------------------------------------------+
mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.c.d');
+----------------------------------------+
| JSON_CONTAINS_PATH(@j, 'one', '$.c.d') |
+----------------------------------------+
|                                      1 |
+----------------------------------------+
mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a.d');
+----------------------------------------+
| JSON_CONTAINS_PATH(@j, 'one', '$.a.d') |
+----------------------------------------+
|                                      0 |
+----------------------------------------+
```

## JSON_CONTAINS
若存在则返回1，不存在则返回0
JSON_CONTAINS(column,value,key),column是指对应的列，value是指key对应的值，key就是键值。
其**查询**的是json串中是否包含"key":"value"这样的**键值对**。
```
SET @test = '{"a":"1","b":"2","c":"3"}';

SELECT json_contains(@test,'"1"','$.a');
返回1
SELECT json_contains(@test,'1','$.a');
返回0
SELECT json_contains(@test,'"2"','$.a');
返回0

```





