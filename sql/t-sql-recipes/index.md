- GO statement is valid for sqlcmd or ssms

- GO statement is a batch operation, and if one GO fail, it will not fail the other goes in a large file

- RETURN statement terminates only the inner sproc if its a nested scenario

- Searched case expression, notice that the case do not need to have an initial expression

```
CASE
   WHEN Boolean_expression_1 THEN result_expression_1
   ...
   WHEN Boolean_expression_n THEN result_expression_n
   ELSE CatchAllValue
END AS ColumnAlias
```

- `COUNT(*)` will count all column, while `COUNT(<NULLABLE COLUMN>)` will return only column which does not have NULL value
- - Minor: COUNT cannot be used on text, image, or ntext datatype
- - Remember there is COUNT_BIG, for BIGINT columns and potentially exceeding integer limit

- DISTINCT can be used within aggregate functions, and will not affect the rows for other columns

- GROUP BY ROLLUP, CUBE, can provide additional level based grouping

- GROUPING SETS is like GROUP BY ROLLUP but using custom group set

```
SELECT  i.Shelf,
        i.LocationID,
        p.Name,
        SUM(i.Quantity) AS Total
FROM    Production.ProductInventory i
        INNER JOIN Production.Product p
          ON i.ProductID = p.ProductID
WHERE   Shelf IN ('A', 'C')
          AND Name IN ('Chain', 'Decal', 'Head Tube')
GROUP BY GROUPING SETS((i.Shelf), (i.Shelf, p.Name), (i.LocationID, p.Name));
```

- GROUP BY ROLLUP can take custom rollup column

- GROUPING function is used to differentiate whether a column participating in a group function is actual NULL or not, the result of the GROUP function can be NULL because data is NULL
- - Go with CUBE, or ROLLUP GROUP BY's

- DISTINCT in a SELECT statement will return unique rows across column selected

- TOP(5) PERCENT is okay too

- ORDER BY of a SELECT .. INTO is not garanteed to be the inserting order

- Identity of a SELECT .. INTO is applied to the new table, except..
- - UNION is used, or identity column is part of an expression, contains a JOIN and a few others

- CROSS APPLY can be used to join table with table-value functions

```
SELECT TOP (5)
        w.WorkOrderID,
        w.OrderQty,
        r.ProductID,
        r.OperationSequence
FROM    Production.WorkOrder w
        CROSS APPLY dbo.fn_WorkOrderRouting(w.WorkOrderID) AS r
ORDER BY w.WorkOrderID,
        w.OrderQty,
        r.ProductID;
```
- CROSS and OUTER APPLY is similar to INNER AND OUTER JOIN

- Actually, the left and right side of the APPLY operator are table sources, so they can be used with anything that returns a table, e.g. including SELECT statements

- TABLESAMPLE extract data page percent, depending on fill factor of pages, the number of rows can varies, and be random

- Everytime when a CTE is reference, the whole query for the CTE is executed
- - That is, CTE does not perform the action once and leave the results available for all references

- If CTE is not the first statement in a batch of statements, then the previous statement must be terminated with a semi-colon

- Terminating SQL statement with semi-colon is part of ANSI specifications
- - Right now SQL Server is not enforcing semi-colon, but future might be

- In recursive CTE, multiple anchor and multiple recursive part can be defined
- - Anchor members can use UNION, UNION ALL, INTERSECT, and EXCEPT

- Use MAXRECURSION to prevent infinite loop, by default its at 100

- DEFAULT can be used with VALUES() table expression for insertion, if there is no default, then NULL will be inserted
- - Max rows is 1000
- - Rows are implicitly casted

- _3_ groups of functions that the OVER clause can be applied to, the aggregate, the ranking, and the analytics
- - Aggregates: avg, checksum_agg, count, count_big, max, min, stdev, stddevp, sum, var, varp
- - Rankings: row_number, rank (same as row_number, except tie will receives same rank, and the next value will skip the ties), dense_rank (same as rank, except no gap on ties, so no skipping), ntile (divides the results into specific number of groups, based on the ordering and optional partition clause)
- - Analytics (2012): ...

- - PARTITION BY is used to restart the calculation when the values in the column changes
- - - Defaulted to the whole table if not specified
- - ORDER BY defines the order in which the OVER clause evaluates the data subset for the function
- - - It can only refers to columns that are in the FROM clause
- - RANGE | ROWS a bit too complicated for now...
