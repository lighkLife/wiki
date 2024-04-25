# Mysql-Use-Case

## 递归查询
```sql
WITH RECURSIVE cte AS (
    SELECT *
    FROM t_sys_resource
    WHERE parent_id = 'root'
    UNION ALL
    SELECT t.*
    FROM t_sys_resource t
             JOIN cte ON t.parent_id = cte.id
)
SELECT id
FROM cte
```