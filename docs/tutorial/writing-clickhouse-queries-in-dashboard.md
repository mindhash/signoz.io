---
id: writing-clickhouse-queries-in-dashboard
title: ClickHouse queries for building dashboards
description: Example clickhouse queries to run analytics on observability data
---

SigNoz gives you ability to write powerful clickhouse queries to plot charts. You can run SQL queries supported by ClickHouse to extract immense value out of your distributed tracing data or logs data. 

Sharing a few examples of some queries which might be helpful in building dashboards.

### GroupBy a tag/attribute in distributed tracing data

```json
select toStartOfInterval(timestamp, INTERVAL 1 MINUTE) AS interval, tagMap['peer.service'] as op_name, toFloat64(avg(durationNano)) as value from signoz_traces.signoz_index_v2  where tagMap['peer.service']!='' and timestamp > now() - INTERVAL 30 MINUTE  group by (op_name, interval) order by (op_name, interval) asc;
```

### Show count of each `customer_id` which is present as attribute of a span event

```json
WITH arrayFilter(x -> JSONExtractString(x, 'name')='Getting customer', events) as filteredEvents
select toStartOfInterval(timestamp, INTERVAL 1 MINUTE) AS interval, toFloat64(count()) as count, arrayJoin(arrayMap(x -> JSONExtractString(JSONExtractString(x, 'attributeMap'), 'customer_id'), filteredEvents)) as resultArray from signoz_traces.signoz_index_v2 where  not empty(filteredEvents) and timestamp > toUnixTimestamp(now() - INTERVAL 30 MINUTE) group by (resultArray, interval) order by (resultArray, interval) asc;
```


### Show sum of values  of `customer_id` which is present as attribute of a span event

```json
WITH arrayFilter(x -> JSONExtractString(x, 'name')='Getting customer', events) as filteredEvents
select toStartOfInterval(timestamp, INTERVAL 1 MINUTE) AS interval, toFloat64(sum(toInt32(resultArray))) as sum, arrayJoin(arrayMap(x -> JSONExtractString(JSONExtractString(x, 'attributeMap'), 'customer_id'), filteredEvents)) as resultArray from signoz_traces.signoz_index_v2 where  not empty(filteredEvents) and timestamp > toUnixTimestamp(now() - INTERVAL 30 MINUTE) group by (resultArray, interval) order by (resultArray, interval) asc;
```


### Plotting a chart on `100ms` interval

Plot a chart of 1 minute showing count of spans in `100ms` interval of service `frontend` with duration > 50ms

```json
select fromUnixTimestamp64Milli(intDiv( toUnixTimestamp64Milli ( timestamp ), 100) * 100) AS interval, toFloat64(count()) as count from (select timestamp from signoz_traces.signoz_index_v2 where serviceName='frontend' and durationNano>=50*exp10(6) and timestamp > now() - INTERVAL 1 MINUTE) group by interval order by interval asc;
```

### Show count of loglines per minute

```json
select toStartOfInterval(fromUnixTimestamp64Nano(timestamp), INTERVAL 1 MINUTE) AS interval, toFloat64(count()) as value from signoz_logs.logs  where timestamp > toUnixTimestamp64Nano(now64() - INTERVAL 30 MINUTE)  group by interval order by interval asc;
```


## Building Alert Queries with Clickhouse data

The most common usecase for alerts is send a notification when a threshold condition (e.g. CPU Usage > 70 or Error count > 10) is met.  Each alert is composed of a threshold condition, timeframe (e.g. Last 5 minutes) and a match condition (e.g. at least once). The rule engine (in SigNoz) is responsible for evaluating the threshold conditions and triggering notifications. 

The threshold condition is further made up of operators (e.g. >), left-hand side (e.g. typically a metric like CPU Usage, Error Count) and the right-hand side (e.g. threshold limits like 70, 10 etc). SigNoz allows you a complete flexibility in choosing all three parameters of the threshold condition.  You can choose a metric data through query builder, promQL or a Clickhouse query.    

The query editor (in the clickhouse tab of alert form) allows you to query Clickhouse to derive metric data and apply a threshold condition on it. The queries can contain the following reserved column aliases and bind variables (for use in WHERE condition) to simplify syntax and allow dynamic parts.  

**Reserved Aliases**: value, res, result, interval 

**Reserved Parameters**: 
- {{.start_datetime}}, {{.end_datetime}} 
- {{.start_timestamp}}, {{.end_timestamp}}
- {{.start_timestamp_ms}}, {{.end_timestamp_ms}}
- {{.start_timestamp_nano}}, {{.end_timestamp_nano}}


### Using Clickhouse queries in the Alert form  

Let's explore writing Clickhouse queries to derive a metric in alert condition. 

A typical SELECT in click house query can have many parameters. But when we write a query for alerts, the rule engine will pick the value from column with alias `value` as metric. 

For example,  The following query retrieves not found errors for a service. In this case, the rule engine will execute the query, select the result from the column with alias `value` and compare it with threshold limit. 

```SELECT count() AS value  
FROM signoz_traces.signoz_error_index_v2 
WHERE (serviceName='api-service’) 
AND (exceptionType='Not found’);
```

If the same query is written without the column alias `value` or `result`, the rule engine will ignore the metric and fail to arrive at a threshold match.  

Second reserved column alias is `interval`.  It is useful when you want to apply threshold check over n time intervals.

```SELECT count() AS value, toStartOfInterval(timestamp, INTERVAL 5 MINUTE) AS interval 
FROM signoz_traces.signoz_error_index_v2 
WHERE (serviceName = 'api-service') 
AND (exceptionType = 'Not found') 
GROUP BY interval
```

The rule engine is also capable of dynamically applying date ranges to your select query.  When you choose `last 15 mins` option in the alert conditions section, the rule engine applies timestamp values in the timeframe condition below.   

```SELECT count() AS value, toStartOfInterval(timestamp, INTERVAL 5 MINUTE) AS interval 
FROM signoz_traces.signoz_error_index_v2 
WHERE timestamp BETWEEN {{.start_datetime}} AND {{.end_datetime}} 
AND (serviceName = 'api-service') 
AND (exceptionType = 'Not found') 
GROUP BY interval
```

The reserved bind variables {{.start_datetime}} and {{.end_datetime}} would allow rule engine to dynamically include date range query when the alert query runs.  





 


