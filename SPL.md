# SPL Cheat Sheet — Splunk Search Processing Language

---

## Search Basics

### Keyword Search

```spl
error                                  | Any event containing "error"
"login failed"                         | Exact phrase match
error OR warning                       | Boolean OR
error NOT timeout                      | Exclude term
(error OR warning) NOT debug           | Grouped booleans
host=web01 status=500                  | Field=value pairs
```

### Wildcards

```spl
fail*                                  | Starts with "fail" (fail, failed, failure)
*exception*                            | Contains "exception"
status=5*                              | Field value starting with 5
```

### Index & Sourcetype Filtering

```spl
index=main sourcetype=syslog           | Narrow to index + sourcetype
index=* NOT index=internal             | All indexes except internal
index=security OR index=network        | Multiple indexes
```

### Piping

```spl
index=web sourcetype=access_combined
| stats count by status
| where count > 100
| sort -count
```

---

## Time Modifiers

### Relative Time Syntax

|Modifier|Meaning|
|---|---|
|`-15m`|15 minutes ago|
|`-1h`|1 hour ago|
|`-1d`|1 day ago|
|`-1w`|1 week ago|
|`-1mon`|1 month ago|
|`-1y`|1 year ago|
|`@h`|Snap to start of hour|
|`@d`|Snap to start of day|
|`@w0`|Snap to start of week (Sunday)|
|`@mon`|Snap to start of month|

### Using earliest and latest

```spl
index=web earliest=-24h latest=now
index=web earliest=-7d@d latest=@d     | Last 7 complete days
index=web earliest=01/01/2024:00:00:00 latest=01/31/2024:23:59:59
```

### strftime tokens

|Token|Meaning|Example|
|---|---|---|
|`%Y`|4-digit year|2024|
|`%m`|Month (01-12)|03|
|`%d`|Day (01-31)|15|
|`%H`|Hour 24h (00-23)|14|
|`%M`|Minutes (00-59)|30|
|`%S`|Seconds (00-59)|45|
|`%s`|Unix epoch|1711065600|

---

## Field Commands

### fields — include or exclude fields

```spl
| fields host, source, status          | Keep only these fields
| fields - _raw, _time                 | Remove these fields
```

### rename

```spl
| rename clientip AS "Client IP" status AS "HTTP Status"
| rename *_count AS *_total            | Wildcard rename
```

### table — format results as a table

```spl
| table _time, host, status, uri_path
```

### sort

```spl
| sort -count                          | Descending by count
| sort +status                         | Ascending by status
| sort -count, +host                   | Multiple fields
| sort 10 -count                       | Top 10 only
```

### dedup — remove duplicate events

```spl
| dedup user                           | First event per user
| dedup user, action                   | Dedup on multiple fields
| dedup 3 user                         | Keep first 3 per user
| dedup user sortby -_time             | Keep most recent
```

### head & tail

```spl
| head 20                              | First 20 results
| tail 10                              | Last 10 results
```

---

## Filtering & Conditionals

### where

```spl
| where status >= 400
| where isnotnull(user)
| where like(uri, "%admin%")
| where match(email, "^[\w.]+@company\.com$")
| where count > avg_count              | Compare two fields
```

### search (inside a pipeline)

```spl
| search status=200 OR status=404
| search user="john*"
```

### if (inline conditional)

```spl
| eval label = if(status >= 400, "Error", "OK")
| eval risk = if(score > 90, "High", if(score > 60, "Medium", "Low"))
```

### case

```spl
| eval severity = case(
    score >= 90, "Critical",
    score >= 70, "High",
    score >= 50, "Medium",
    true(), "Low"
  )
```

---

## Stats & Aggregation

### stats

```spl
| stats count                          | Total event count
| stats count by host                  | Count grouped by field
| stats count, avg(duration) by host   | Multiple aggregations
| stats dc(user) AS unique_users       | Distinct count
| stats sum(bytes) AS total_bytes by src_ip
| stats min(response_time), max(response_time), avg(response_time) by endpoint
| stats values(status) AS statuses by host    | All unique values as multivalue
| stats list(uri) AS urls by user             | All values (including dupes)
| stats count AS events, sum(bytes) AS volume by host, sourcetype
```

### Common Stats Functions

|Function|Description|Example|
|---|---|---|
|`count`|Event count|`count`|
|`count(field)`|Non-null count|`count(status)`|
|`dc(field)`|Distinct count|`dc(user)`|
|`sum(field)`|Sum|`sum(bytes)`|
|`avg(field)`|Average|`avg(duration)`|
|`min(field)`|Minimum|`min(response_ms)`|
|`max(field)`|Maximum|`max(response_ms)`|
|`median(field)`|Median|`median(score)`|
|`stdev(field)`|Std deviation|`stdev(latency)`|
|`perc95(field)`|95th percentile|`perc95(response_time)`|
|`values(field)`|Unique values (MV)|`values(status)`|
|`list(field)`|All values (MV)|`list(url)`|
|`first(field)`|First seen|`first(action)`|
|`last(field)`|Last seen|`last(action)`|
|`range(field)`|max - min|`range(bytes)`|

### eventstats — add stats as new fields without collapsing

```spl
| eventstats avg(duration) AS global_avg by host
| eval above_avg = if(duration > global_avg, "yes", "no")
```

### streamstats — running/rolling calculations

```spl
| sort _time
| streamstats count AS row_num
| streamstats sum(bytes) AS running_total by src_ip
| streamstats window=5 avg(response_time) AS rolling_5min_avg
```

---

## Transforming Commands

### timechart — time-based charting

```spl
| timechart count                             | Events over time
| timechart count by status                  | Events by field over time
| timechart span=1h avg(response_time)       | 1-hour buckets
| timechart span=1d sum(bytes) by host limit=5  | Top 5 hosts per day
```

### chart

```spl
| chart count over host by status            | Rows=host, columns=status
| chart avg(duration) by endpoint, status
| chart count over _time span=1h by sourcetype
```

### top — most common values

```spl
| top user                            | Top 10 users by default
| top limit=5 uri
| top limit=0 host                    | All values (no limit)
| top clientip by status              | Top per group
| top showperc=f showcount=t user     | Hide percentage
```

### rare

```spl
| rare user                           | Least common users
| rare limit=5 action by host
```

### iplocation — GeoIP enrichment

```spl
| iplocation src_ip
| table src_ip, City, Country, lat, lon
```

---

## Eval & Calculated Fields

### eval basics

```spl
| eval field_name = expression
| eval upper_status = upper(status)
| eval duration_sec = duration / 1000
| eval full_name = first_name . " " . last_name    | String concat
| eval is_error = if(status >= 400, 1, 0)
```

### Arithmetic

```spl
| eval total = price * quantity
| eval discount = total * 0.1
| eval net = total - discount
| eval ratio = hits / (hits + misses) * 100
```

### Multiple evals in one

```spl
| eval
    size_kb = bytes / 1024,
    size_mb = bytes / 1048576,
    label = if(status >= 400, "Error", "OK")
```

### Chaining evals

```spl
| eval size_kb = bytes / 1024
| eval size_label = size_kb . " KB"
```

### Type conversion

```spl
| eval str_val = tostring(count)
| eval num_val = tonumber(string_field)
| eval bytes_hr = tostring(bytes, "commas")         | "1,234,567"
| eval file_size = tostring(bytes, "bytes")          | "1.2 MB"
| eval duration_str = tostring(secs, "duration")    | "1m 30s"
```

---

## String Functions

```spl
| eval x = upper("hello")               | "HELLO"
| eval x = lower("HELLO")               | "hello"
| eval x = len("hello")                 | 5
| eval x = substr("hello world", 1, 5)  | "hello"  (1-indexed)
| eval x = trim("  hello  ")            | "hello"
| eval x = ltrim("  hello")             | "hello"
| eval x = rtrim("hello  ")             | "hello"
| eval x = replace(field, "old", "new") | Replace all occurrences
| eval x = split(field, ",")            | Splits to multivalue
| eval x = mvjoin(mv_field, ", ")       | Join multivalue to string
| eval x = like(field, "err%")          | 1 if match, else 0
| eval x = match(field, "^\d+$")        | Regex match
| eval x = urldecode(encoded_url)
| eval x = urlencode(url)
| eval x = md5(field)
| eval x = sha256(field)
```

---

## Mathematical Functions

```spl
| eval x = abs(-5)            | 5
| eval x = ceiling(4.1)       | 5
| eval x = floor(4.9)         | 4
| eval x = round(3.14159, 2)  | 3.14
| eval x = sqrt(16)           | 4
| eval x = pow(2, 10)         | 1024
| eval x = exp(1)             | 2.718...
| eval x = log(100, 10)       | 2
| eval x = ln(100)            | Natural log
| eval x = pi()               | 3.14159...
| eval x = random()           | Random integer
```

---

## Date & Time Functions

### now, time, and epoch

```spl
| eval now_epoch = now()               | Current Unix timestamp
| eval now_str = strftime(now(), "%Y-%m-%d %H:%M:%S")
| eval _time_str = strftime(_time, "%Y-%m-%d")
| eval parsed_time = strptime("2024-03-15", "%Y-%m-%d")
```

### Date math

```spl
| eval age_seconds = now() - _time
| eval age_days = (now() - _time) / 86400
| eval yesterday = now() - 86400
```

### Bucketing time

```spl
| eval hour_bucket = strftime(_time, "%Y-%m-%d %H:00")
| eval day_bucket  = strftime(_time, "%Y-%m-%d")
| eval week_bucket = strftime(_time, "%Y-W%W")
```

### bin (time & numeric bucketing)

```spl
| bin _time span=1h              | Bucket into 1-hour bins
| bin _time span=15m
| bin response_time span=100     | Bucket numeric field
| bin bytes span=1048576 AS size_bucket
```

---

## Lookup & Enrichment

### lookup

```spl
| lookup userdb.csv user_id OUTPUT username, department
| lookup threat_intel.csv src_ip OUTPUT threat_score, category
| lookup geo_data.csv country_code AS src_country OUTPUT region
```

### inputlookup — search a lookup file

```spl
| inputlookup userdb.csv
| inputlookup userdb.csv WHERE department="Engineering"
```

### outputlookup — write results to a lookup file

```spl
| stats count by src_ip
| outputlookup ip_counts.csv
```

### Automatic Lookups

Configured in Settings > Lookups > Automatic Lookups — applied transparently on search.

---

## Subsearches

Subsearches run first and return a filter or value set for the outer search.

### Basic subsearch

```spl
index=web
  [search index=alerts type=critical | fields src_ip]
```

### Subsearch to generate a field filter

```spl
index=web [
  search index=auth action=failed
  | stats count by user
  | where count > 10
  | fields user
]
```

### format command with subsearch

```spl
index=web [
  search index=threat | fields src_ip | format
]
```

> **Limits:** Subsearches return max 10,000 results by default; they have a 60-second timeout. Prefer `join` or `lookup` for large datasets.

---

## Transactions

Groups related events into a single transaction event.

```spl
| transaction session_id                        | Group by session
| transaction user maxspan=30m                  | Max 30-min session
| transaction user maxpause=5m                  | Break on 5m gap
| transaction session_id startswith="login" endswith="logout"
| transaction host maxevents=50                 | Limit events per txn
```

### Useful transaction fields

|Field|Description|
|---|---|
|`duration`|Time from first to last event|
|`eventcount`|Number of events in transaction|
|`closed_txn`|1 if transaction is closed|

```spl
| transaction session_id
| where duration > 300 AND eventcount > 5
| stats avg(duration), avg(eventcount) by user
```

---

## Joins

### join

```spl
| join user [search index=hr | table user, department, manager]
| join type=left src_ip [search index=geoip | table ip, country]
| join type=outer user [search index=auth | table user, last_login]
```

> **Note:** `join` is expensive. Prefer `lookup` for static data and `tstats`/`stats` for large datasets.

---

## Charting & Visualization

### Single Value

```spl
index=web | stats count AS "Total Requests"
```

### Line / Area Chart

```spl
index=web | timechart span=1h count by status
```

### Bar Chart

```spl
index=web | stats count by host | sort -count
```

### Pie Chart

```spl
index=web | stats count by status | sort -count | head 5
```

### Heatmap (using chart)

```spl
index=web
| eval hour = strftime(_time, "%H")
| eval day  = strftime(_time, "%A")
| chart count over day by hour
```

### Trellis / Split by field

```spl
index=web | timechart count by host
```

---

## Alerting & Scheduling

Saved searches used as alerts — configured in the UI under Alerts, or via the `schedule` setting in `savedsearches.conf`.

### Alert trigger conditions

```spl
| stats count | where count > 1000               | Number of results
| stats count | where count = 0                  | No results (gap detection)
```

### Throttling in SPL context

```spl
index=security action=failed_login
| stats count by src_ip
| where count > 20
| eval message = "Brute force suspected from " . src_ip
```

---

## Macros & Saved Searches

### Calling a macro

```spl
`error_events`                         | No-arg macro
`threshold_filter(500)`                | Macro with one arg
`date_range(2024-01-01, 2024-03-31)`   | Macro with two args
```

### Macro definition (Settings > Advanced Search > Macros)

```
Name:       error_events
Definition: index=main (error OR exception OR critical)
```

```
Name:       threshold_filter(1)
Arguments:  threshold
Definition: | where count > $threshold$
```

---

## Rex & Regex Extraction

### rex — extract fields with named groups

```spl
| rex "(?<method>GET|POST|PUT|DELETE) (?<uri>\S+)"
| rex field=_raw "user=(?<username>\w+)"
| rex "status=(?<status_code>\d{3})"
```

### rex mode=sed — replace/substitute

```spl
| rex mode=sed field=message "s/password=[^ ]*/password=REDACTED/g"
| rex mode=sed field=email "s/@.*$//"
```

### spath — extract from JSON/XML

```spl
| spath                                       | Auto-extract all paths
| spath input=_raw path=user.email
| spath input=json_field "events{}.type"
| spath input=_raw path="items{}.price"
```

### erex — example-based regex extraction

```spl
| erex myfield examples="192.168.1.1, 10.0.0.1"
```

---

## Multivalue Fields

```spl
| eval tags = split("red,green,blue", ",")     | Create multivalue
| eval first_tag = mvindex(tags, 0)             | First element (0-indexed)
| eval last_tag = mvindex(tags, -1)             | Last element
| eval tag_count = mvcount(tags)
| eval joined = mvjoin(tags, " | ")             | Join to string
| eval filtered = mvfilter(match(tags, "^r"))   | Filter elements
| eval deduped = mvdedup(tags)
| eval sorted = mvsort(tags)
| eval appended = mvappend(tags, "purple")

| mvexpand tags                                 | One row per value
```

### stats on multivalue

```spl
| stats values(status) AS all_statuses by host
| stats dc(values(status)) AS unique_statuses by host
```

---

## Performance Tips

### Put fast, narrowing filters first

```spl
| index=web host=web01 status=500              | Good — indexed fields first
| index=web | where host="web01" | search status=500  | Slower — filtering late
```

### Use tstats for metric-level speed

```spl
| tstats count WHERE index=web BY host, status  | Extremely fast (index metadata only)
| tstats sum(bytes) AS total WHERE index=web BY _time span=1h
```

### Avoid wildcards at start of strings

```spl
status=500*                   | OK — wildcard at end
status=*500                   | Slow — wildcard at start
```

### Limit fields early

```spl
| fields host, status, bytes  | Drop unwanted fields early in pipeline
```

### Use `summariesonly` with data models

```spl
| tstats summariesonly=t count FROM datamodel=Web.Web BY Web.status
```

### Accelerate with data models

- Build CIM-compliant data models
- Enable acceleration (Settings > Data Models > Edit Acceleration)
- Query with `tstats` or `pivot`

### Summary indexes

```spl
| stats count by host | collect index=summary_idx sourcetype=host_counts
```

---

## Quick Reference Card

### Most Used Commands

|Command|Purpose|
|---|---|
|`stats`|Aggregate events|
|`eval`|Create/transform fields|
|`where`|Filter rows|
|`table`|Select columns|
|`sort`|Order results|
|`dedup`|Remove duplicates|
|`rex`|Regex extraction|
|`timechart`|Time series aggregation|
|`lookup`|Enrich with CSV/KV store|
|`transaction`|Group related events|
|`join`|Combine search results|
|`spath`|Parse JSON/XML|
|`top`|Most frequent values|
|`rare`|Least frequent values|
|`bin`|Bucket values|
|`fields`|Include/exclude fields|
|`rename`|Rename fields|
|`head`/`tail`|Limit results|
|`mvexpand`|Expand multivalue fields|
|`tstats`|Ultra-fast metadata search|

---

_SPL Cheat Sheet — Splunk Enterprise / Splunk Cloud_