---
title: sql基础
---

sql基础知识。

# 联合查询

1\. 联合查询并插入新表 :

    select a.*, b.name as namespace_name, b.version_format into tmp from public.vulnerability a left join public.namespace b on a.namespace_id=b.id order by a.id asc 

2\. 多表联合查询 :

    select * from vulnerability aa left join (select * from feature a left join featureversion b on a.id=b.feature_id) as tmp on aa.namespace_id=tmp.namespace_id order by aa.id asc limit 1

    # 四个表格联合查询
    select * from vulnerability a left join namespace b on a.namespace_id=b.id left join feature c on b.id=c.namespace_id left join featureversion d on c.id=d.feature_id order by a.id asc limit 6000

::: {.warning}
::: {.title}
Warning
:::

left
join如果on条件的值存在重复值，可能会导致查出来的数据量大于左表，所以on条件的字段请确保唯一。
:::
