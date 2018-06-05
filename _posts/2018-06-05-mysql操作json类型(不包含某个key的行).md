---
layout: post
title: mysql操作json类型(不包含某个key的行)
key: szk
tags: MySql
---

## 查找json列不包含某个键的行

假设有一数据表名为resource_obj,其中列json_attrs为当传来的aclNumber不为空时，查出来的是

```
<choose>
	<when test="aclNumber != null and aclNumber != ''">
		and JSON_EXTRACT(resource_obj.json_attrs,'$.acl_number') = #{aclNumber,jdbcType=VARCHAR}
	</when>
	<otherwise>
		and JSON_CONTAINS_PATH(resource_obj.json_attrs,'one','$.acl_number') = '0'
	</otherwise>
</choose>

```

