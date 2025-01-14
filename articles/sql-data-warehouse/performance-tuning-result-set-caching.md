---
title: Performance tuning with result set caching | Microsoft Docs
description: Feature overview  
services: sql-data-warehouse
author: XiaoyuMSFT
manager: craigg 
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: development
ms.date: 10/10/2019
ms.author: xiaoyul
ms.reviewer: nidejaco;  
---

# Performance tuning with result set caching  
When result set caching is enabled, Azure SQL Data Warehouse automatically caches query results in the user database for repetitive use.  This allows subsequent query executions to get results directly from the persisted cache so recomputation is not needed.   Result set caching improves query performance and reduces compute resource usage.  In addition, queries using cached results set do not use any concurrency slots and thus do not count against existing concurrency limits. For security, users can only access the cached results if they have the same data access permissions as the users creating the cached results.  

## Key commands
[Turn ON/OFF result set caching for a user database](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azure-sqldw-latest)

[Turn ON/OFF result set caching for a session](https://docs.microsoft.com/sql/t-sql/statements/set-result-set-caching-transact-sql?view=azure-sqldw-latest)

[Check the size of cached result set](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-showresultcachespaceused-transact-sql?view=azure-sqldw-latest)  

[Clean up the cache](https://docs.microsoft.com/sql/t-sql/database-console-commands/dbcc-dropresultsetcache-transact-sql?view=azure-sqldw-latest)

## What's not cached  

Once result set caching is turned ON for a database, results are cached for all queries until the cache is full, except for these queries:
- Queries using non-deterministic functions such as DateTime.Now()
- Queries using user defined functions
- Queries returning data with row size larger than 64KB

Queries with large result sets (for example, > 1 million rows) may experience slower performance during the first run when the result cache is being created.

## When cached results are used

Cached result set is reused for a query if all of the following requirements are all met:
- The user who's running the query has access to all the tables referenced in the query.
- There is an exact match between the new query and the previous query that generated the result set cache.
- There is no data or schema changes in the tables where the cached result set was generated from.

Run this command to check if a query was executed with a result cache hit or miss. If there is a cache hit, the result_cache_hit will return 1.

```sql
SELECT request_id, command, result_cache_hit FROM sys.pdw_exec_requests 
WHERE request_id = <'Your_Query_Request_ID'>
```

## Manage cached results 

The maximum size of result set cache is 1 TB per database.  The cached results are automatically invalidated when the underlying query data change.  

The cache eviction is managed by Azure SQL Data Warehouse automatically following this schedule: 
- Every 48 hours if the result set hasn't been used or has been invalidated. 
- When the result set cache approaches the maximum size.

Users can manually empty the entire result set cache by using one of these options: 
- Turn OFF the result set cache feature for the database 
- Run DBCC DROPRESULTSETCACHE while connected to the database

Pausing a database won't empty cached result set.  

## Next steps
For more development tips, see [SQL Data Warehouse development overview](sql-data-warehouse-overview-develop.md). 
