# Chapter 44

- sys.dm_exec_query_plan, for example, get most CPU intensive SQL

```
select top 10  
    sum(qs.total_worker_time) as total_cpu_time,  
    sum(qs.execution_count) as total_execution_count,

    qs.plan_handle, st.text  
from  
    sys.dm_exec_query_stats qs
cross apply sys.dm_exec_sql_text(qs.plan_handle) as st
group by qs.plan_handle, st.text
order by sum(qs.total_worker_time) desc
```

- `showplan` directive can return the estimated query plan as a message or result set

- `statistics profile` directive can return the actual query plan

- The Query Optimizer (QO) relies heavily on statistics.

- The difference between the testimated and the actual plan isn't the plan itself, the sequence of physical operations are often the same
- - The difference are the number of rows returned by each operator

- Tips: read the cost as cost x 1000, i.e. 0.005 as 5, 0.325 as 325. As they are relative values

- DMVs that expose the query execution plans
- - `sys.dm_exec_cached_plans` plan type, memory size, usecounts
- - `sys.dm_exec_query_stats` aggregate execution statistics (e.g. last_execution_time, max_elapsed_time)
- - `sys.dm_exec_requests` plans that are currently executing
- - `sys.dm_exec_procedure_stats` aggregated execution statistics for sproc

- Plan handle is a binary identifier of the query execution plan in memory
- - It can be passed to some dynamic management functions with a `cross apply` to extract the query text or query execution plan
- - `sys.dm_exec_query_plan(plan_handle)` the query execution plan in XML, might fail for XML nesting larger than 128
- - `sys.dm_exec_text_query_plan(plan_handle)` the query execution plan as a text showplan
- - `sys.dm_exec_sql_text(plan_handle)` the query SQL statement

- When defining clustered index without using unique keyword, a "uniquifier" is added to the clustered key to ensure the set of values is unique
- - This uniqifier will be stored in NCI as well, making indexes larger overall

- Logically, the CI pages are maintained in the CI sort order
- - Physically, pages are connected via LLs (linked lists), so their physical order is not garanteed to be in order
- - Rows on a page are also not garanteed to by stored in sorted order

- If NCI is unique, then only leaf node contains pointer to base table
- - If NCI is not unique, then all level contain a pointer to the base table
- - If base table is CI, then the clustered keys are stored in NCI
- - If table is a heap, then NCI contains row-identifier for the base table record

- Composite key should do search from column left to right, otherwise it cannot take advantage of the index

- Difference between unique constraints and primary key is unique constrains can contain 1 single NULL value, while PK cannot be null
- - Otherwise unique constraints also builds an unique index

- Page split is costly, can insert lots of data in the transaction log, leave the data fragmented hence it cannot be read in a single continguous way

- DBCC SHOW_STATISTICS ('dbo.Table', IX_INDEXNAME), shows the index
- - High density makes poor index, while low density is good and means the data is more selective
- - Changing order of a composite key might improve selectivity, but becareful that some query might depends on the order

- Row identifier is composed of FileID:PageNum:SlotNum
- - Row id cannot be directly queried
