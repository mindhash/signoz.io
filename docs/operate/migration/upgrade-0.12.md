---
id: upgrade-0.12
title: Upgrade to 0.12
sidebar_label: Upgrade to 0.12
---

# Features of this release 
v0.12 introduces distributed clickhouse setup. 


# After upgrading to v0.12
- All the tables in clickhouse have been prefixed with distributed_*. In case you have used clickhouse queries in dashboard or alerts, you would have to update the queries with the new table names. The old table names will continue to work for single node installation but if you plan to configure distributed setup in future, we recommend changing table names at the earliest. 


