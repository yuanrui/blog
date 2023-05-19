---
title: Oracle常用sql(第一部分)
date: 2016-07-20 11:44:13
tags:
	- Oracle
  - Sql
  - 开发笔记
  - 读书笔记
categories:
  - Sql
  - Oracle
---
#### 查看表空间使用情况
```
SELECT
  a.TABLESPACE_NAME,
  a.BYTES / 1024 / 1024 "Sum MB",
  (a.BYTES - b.BYTES) / 1024 / 1024 "Used MB",
  b.BYTES / 1024 / 1024 "free MB",
  ROUND(((a.BYTES - b.BYTES) / a.BYTES) * 100, 2) 
  AS "percent_used"
FROM  (SELECT
        TABLESPACE_NAME,
        SUM(BYTES) bytes
      FROM DBA_DATA_FILES
      GROUP BY TABLESPACE_NAME) a,
      (SELECT
        TABLESPACE_NAME,
        SUM(BYTES) bytes,
        MAX(BYTES) largest
      FROM DBA_FREE_SPACE
      GROUP BY TABLESPACE_NAME) b
WHERE a.TABLESPACE_NAME = b.TABLESPACE_NAME
ORDER BY ((a.BYTES - b.BYTES) / a.BYTES) DESC
```
#### 查询数据库高速缓冲区命中率
相关计算公式：1-('physical reads cache'/('db block gets from cache' + 'consistent gets from cache'))
```
select name, value from v$sysstat
where name in ('db block gets from cache', 'consistent gets from cache', 'physical reads cache');
```
#### 查询命中率比较低的Sql(低效Sql)
```
SELECT
  PARSING_SCHEMA_NAME,
  EXECUTIONS,
  DISK_READS,
  BUFFER_GETS,
  ROUND((BUFFER_GETS - DISK_READS) / BUFFER_GETS, 2) Hit_radio,
  ROUND(DISK_READS / EXECUTIONS, 2) Reads_per_run,
  SQL_TEXT
FROM V$SQLAREA
WHERE EXECUTIONS > 0
AND BUFFER_GETS > 0
AND (BUFFER_GETS - DISK_READS) / BUFFER_GETS < 0.8
ORDER BY 4 DESC;
```
#### 找出总消耗时间最多的前10条语句
```
SELECT sql_id,child_number,sql_text, elapsed_time 
  FROM (SELECT sql_id, child_number, sql_text, elapsed_time, cpu_time,
               disk_reads,
               RANK () OVER (ORDER BY elapsed_time DESC) AS elapsed_rank
          FROM v$sql)
 WHERE elapsed_rank <= 10
```
#### 按等待时间排序等待事件
```
SELECT   wait_class, event, total_waits AS waits,
         ROUND (time_waited_micro / 1000) AS total_ms,
         ROUND (time_waited_micro * 100 / SUM (time_waited_micro) OVER (),
                2
               ) AS pct_time,
         ROUND ((time_waited_micro / total_waits) / 1000, 2) AS avg_ms
    FROM v$system_event
   WHERE wait_class <> 'Idle'
ORDER BY time_waited_micro DESC;
```
#### 查询会话历史表中锁等待的Sql和对象
(最近一个小时左右), 可以将v$active_session_history替换为dba_hist_active_sess_history以显示更长时间范围内的等待对象。
```
WITH ash_query AS (
     SELECT substr(event,6,2) lock_type,program, 
            h.module, h.action,   object_name,
            SUM(time_waited)/1000 time_ms, COUNT( * ) waits, 
            username, sql_text,
            RANK() OVER (ORDER BY SUM(time_waited) DESC) AS time_rank,
            ROUND(SUM(time_waited) * 100 / SUM(SUM(time_waited)) 
                OVER (), 2)             pct_of_time
      FROM  v$active_session_history h 
      JOIN  dba_users u  USING (user_id)
      LEFT OUTER JOIN dba_objects o
           ON (o.object_id = h.current_obj#)
      LEFT OUTER JOIN v$sql s USING (sql_id)
     WHERE event LIKE 'enq: %'
     GROUP BY substr(event,6,2) ,program, h.module, h.action, 
         object_name,  sql_text, username)
SELECT lock_type,module, username,  object_name, time_ms,pct_of_time,
         sql_text
FROM ash_query
WHERE time_rank < 11
ORDER BY time_rank;
```
#### 查询遭遇最多行级锁等待的数据库对象
```
SELECT object_name, VALUE row_lock_waits, 
       ROUND(VALUE * 100 / SUM(VALUE) OVER (), 2) pct
  FROM v$segment_statistics
 WHERE statistic_name = 'row lock waits' AND VALUE > 0
 ORDER BY VALUE DESC;
```
#### 查询锁的持有者和等待获取锁的会话
```
 WITH sessions AS 
       (SELECT /*+ materialize*/ username,sid,sql_id
          FROM v$session),
     locks AS 
        (SELECT /*+ materialize */ *
           FROM v$lock)
SELECT l2.type,s1.username blocking_user, s1.sid blocking_sid, 
        s2.username blocked_user, s2.sid blocked_sid, sq.sql_text
  FROM locks l1
  JOIN locks l2 USING (id1, id2)
  JOIN sessions s1 ON (s1.sid = l1.sid)
  JOIN sessions s2 ON (s2.sid = l2.sid)
  LEFT OUTER JOIN  v$sql sq
       ON (sq.sql_id = s2.sql_id)
 WHERE l1.BLOCK = 1 AND l2.request > 0
```
#### 查询消耗PGA内存最多的5个进程
查询消耗PGA内存最多的5个进程和当前正在执行的Sql
```
WITH pga AS 
    (SELECT sid,
            ROUND(SUM(CASE name WHEN 'session pga memory' 
                       THEN VALUE / 1048576 END),2) pga_memory_mb,
            ROUND(SUM(CASE name WHEN 'session pga memory max' 
                      THEN VALUE / 1048576  END),2) max_pga_memory_mb
      FROM v$sesstat  
      JOIN v$statname  USING (statistic#)
     WHERE name IN ('session pga memory','session pga memory max' )
     GROUP BY sid)
SELECT sid, username,s.module, 
       pga_memory_mb, 
       max_pga_memory_mb, substr(sql_text,1,70) sql_text
  FROM v$session s
  JOIN (SELECT sid, pga_memory_mb, max_pga_memory_mb,
               RANK() OVER (ORDER BY pga_memory_mb DESC) pga_ranking
         FROM pga)
  USING (sid)
  LEFT OUTER JOIN v$sql sql 
    ON  (s.sql_id=sql.sql_id and s.sql_child_number=sql.child_number)
 WHERE pga_ranking <=5
 ORDER BY  pga_ranking
```
#### 合并PGA+SGA的内存顾问适用于11g
Combined (PGA+SGA) memory advice report for 11g 
```
WITH db_cache_times AS 
    (SELECT current_size current_cache_mb, 
            size_for_estimate target_cache_mb,
            (estd_physical_read_time - current_time) 
               cache_secs_delta
       FROM v$db_cache_advice,
            (SELECT size_for_estimate current_size,
                    estd_physical_read_time current_time
               FROM v$db_cache_advice
              WHERE  size_factor = 1
                AND name = 'DEFAULT' AND block_size = 8192)
       WHERE name = 'DEFAULT' AND block_size = 8192),
 pga_times AS 
     (SELECT current_size / 1048576 current_pga_mb,
             pga_target_for_estimate / 1048576 target_pga_mb,
             estd_time-base_time pga_secs_delta 
        FROM v$pga_target_advice , 
             (SELECT pga_target_for_estimate current_size,
                     estd_time base_time
                FROM v$pga_target_advice 
               WHERE pga_target_factor = 1))
SELECT current_cache_mb||'MB->'||target_cache_mb||'MB' Buffer_cache,
       current_pga_mb||'->'||target_pga_mb||'MB' PGA,
       pga_secs_delta,cache_secs_delta,
       (pga_secs_delta+cache_secs_delta) total_secs_delta
  FROM db_cache_times d,pga_times p
 WHERE (target_pga_mb+target_cache_mb)
        <=(current_pga_mb+current_cache_mb)
   AND (pga_secs_delta+cache_secs_delta) <0
 ORDER BY (pga_secs_delta+cache_secs_delta);
```
执行结果
BUFFER_CACHE                                                                     PGA                                                                              PGA_SECS_DELTA CACHE_SECS_DELTA TOTAL_SECS_DELTA
-------------------------------------------------------------------------------- -------------------------------------------------------------------------------- -------------- ---------------- ----------------
1760MB->2288MB                                                                   1120->560MB                                                                                   0            -2008            -2008
1760MB->2112MB                                                                   1120->560MB                                                                                   0            -1612            -1612
1760MB->1936MB                                                                   1120->560MB                                                                                   0             -901             -901
1760MB->1936MB                                                                   1120->840MB                                                                                   0             -901             -901
结果分析：给高速缓存区增加528MB(2288MB-1760MB)内存可以节约2008秒的时间。减少PGA内存560MB(1120MB-560MB)未受到影响。

#### 查询显示PGA内存管理方式
```
--alter system set workarea_size_policy=manual; --将pga内存管理设置为手动管理
--alter system set workarea_size_policy=auto;   --将pga内存管理设置为自动管理
show parameter workarea_size_policy;
```
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
workarea_size_policy                 string      AUTO
值为AUTO时表示自动管理，为MANUAL是表示手动管理

#### 查询数据库中子表上没有索引的外键
```
SELECT c.owner,
         c.constraint_name,
         c.table_name,
         cc.column_name,
         c.status
    FROM dba_constraints c, dba_cons_columns cc
   WHERE c.constraint_type = 'R'
         AND c.owner NOT IN
                ('SYS',
                 'SYSTEM',
                 'SYSMAN',
                 'EXFSYS',
                 'WMSYS',
                 'OLAPSYS',
                 'OUTLN',
                 'DBSNMP',
                 'ORDSYS',
                 'ORDPLUGINS',
                 'MDSYS',
                 'CTXSYS',
                 'AURORA$ORB$UNAUTHENTICATED',
                 'XDB',
                 'FLOWS_030000',
                 'FLOWS_FILES')
         AND c.owner = cc.owner
         AND c.constraint_name = cc.constraint_name
         AND NOT EXISTS
                    (SELECT 'x'
                       FROM dba_ind_columns ic
                      WHERE     cc.owner = ic.table_owner
                            AND cc.table_name = ic.table_name
                            AND cc.column_name = ic.column_name
                            AND cc.position = ic.column_position
                            AND NOT EXISTS
                                       (SELECT owner, index_name
                                          FROM dba_indexes i
                                         WHERE     i.table_owner = c.owner
                                               AND i.index_Name = ic.index_name
                                               AND i.owner = ic.index_owner
                                               AND (i.status = 'UNUSABLE'
                                                    OR i.partitioned = 'YES'
                                                       AND EXISTS
                                                              (SELECT 'x'
                                                                 FROM dba_ind_partitions ip
                                                                WHERE status =
                                                                         'UNUSABLE'
                                                                      AND ip.
                                                                           index_owner =
                                                                             i.
                                                                              owner
                                                                      AND ip.
                                                                           index_Name =
                                                                             i.
                                                                              index_name
                                                               UNION ALL
                                                               SELECT 'x'
                                                                 FROM dba_ind_subpartitions isp
                                                                WHERE status =
                                                                         'UNUSABLE'
                                                                      AND isp.
                                                                           index_owner =
                                                                             i.
                                                                              owner
                                                                      AND isp.
                                                                           index_Name =
                                                                             i.
                                                                              index_name))))
ORDER BY 1, 2
```
