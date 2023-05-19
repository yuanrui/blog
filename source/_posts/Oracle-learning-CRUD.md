---
title: Oracle学习笔记-CRUD
date: 2017-02-08 10:39:00
tags:
  - Oracle
  - Sql
  - 开发笔记
categories:
  - Sql
  - Oracle
---
### Update statement
假设有如下表(T_SCORE_LEVEL)

| ID | SCORE | LEVEL | 
|:------------- |:------------- |
| 1 | 60 | 0 |
| 2 | 70 | 0 |
| 3 | 75 | 0 |
| 4 | 65 | 0 |
| 5 | 95 | 0 |
| 6 | 82 | 0 |

需要更新LEVEL字段，更新规则:
Score大于等于90小于等于100，Level=1;
Score大于等于80小于等于90，Level=2;
Score大于等于70小于等于80，Level=3;
Score大于等于60小于等于70，Level=4;
Score小于60，Level=5;
请写出对应的sql

```
UPDATE T_SCORE_LEVEL
SET LEVEL = (
	CASE 
		WHEN SCORE >= 90 AND SCORE <= 100 THEN 1
		WHEN SCORE >= 80 AND SCORE < 90 THEN 2
		WHEN SCORE >= 70 AND SCORE < 80 THEN 3
		WHEN SCORE >= 60 AND SCORE < 70 THEN 4
		WHEN SCORE < 60 THEN 5
);

COMMIT
/
```

