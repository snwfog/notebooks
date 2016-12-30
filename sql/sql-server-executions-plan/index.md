- DMOs `sys.dm_exec_query_plan` can be used to retrieve cached plan

```
  SELECT [cp].[refcounts],
         [cp].[usecounts],
         [cp].[objtype],
         [st].[dbid],
         [st].[objectid],
         [st].[text],
         [qp].[query_plan]
  FROM   sys.dm_exec_cached_plans cp
         CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) st
         CROSS APPLY sys.dm_exec_query_plan(cp.plan_handle) qp;
```

- - The main purpose of cross-join is to enable table functions with parameters to be executed once per row and then joined to the results. (http://stackoverflow.com/questions/1139160/when-should-i-use-cross-apply-over-inner-join)
- - Above sql use the plan handle to retrieve additional information
