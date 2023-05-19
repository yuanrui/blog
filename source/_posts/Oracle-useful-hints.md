---
title: Oracle常用提示
date: 2016-11-22 09:55:22
tags:
  - Oracle
  - Sql
  - 开发笔记
  - 读书笔记
categories:
  - Sql
  - Oracle
toc: false 
---
取前N条分页数据提示
采用row_number进行高效分页
参考：http://www.oracle.com/technetwork/issue-archive/2007/07-jan/o17asktom-093877.html
示例
```
select *
 from (
select /*+ first_rows(25) */
  your_columns,
  row_number() 
  over (order by something unique)rn
 from your_tables )
where rn between :n and :m 
order by rn; 
```

结果集缓存提示
用于缓存长期稳定的小表数据
参考：http://www.oracle.com/technetwork/articles/sql/11g-caching-pooling-088320.html
示例
```
SELECT /*+ result_cache */ * FROM Your_Tables;
```

