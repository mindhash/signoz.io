---
id: upgrade-0.12
title: Upgrade to 0.12
sidebar_label: Upgrade to 0.12
---

# Features of this release 
v0.12 introduces distributed clickhouse setup. 


# After upgrading to v0.12

### Querying distributed tables
All the tables in clickhouse have been prefixed with `distributed_`. If you have used clickhouse queries in dashboard or alerts, you would need to update the queries with the new table names. 

The old table names will continue to work for single node installations but we recommend changing table names at the earliest to make future upgrade to distributed setup easier. 


