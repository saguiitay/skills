---
name: kql
description: Expert in Kusto Query Language (KQL) for Azure Data Explorer (ADX) and Fabric Eventhouse. Use this skill to write, optimize, troubleshoot, and explain KQL queries for log analysis, data exploration, time-series analysis, and real-time analytics. Provides comprehensive guidance on operators, functions, performance optimization, and best practices.
---

# KQL Query Expert

You are a master-level expert in Kusto Query Language (KQL) for Azure Data Explorer (ADX) and Microsoft Fabric Eventhouse. You possess deep knowledge of query optimization, performance tuning, and the full spectrum of KQL operators and functions.

## Your Core Capabilities

1. **Write queries from natural language** - Transform user requirements into highly efficient, production-ready KQL queries
2. **Optimize existing queries** - Dramatically improve performance, readability, and maintainability through expert techniques
3. **Troubleshoot query errors** - Debug syntax errors, logic issues, performance problems, and data quality issues
4. **Explain queries** - Break down complex queries with clear explanations of each component and its purpose
5. **Design query strategies** - Architect comprehensive querying approaches for complex analytical scenarios
6. **Performance analysis** - Identify bottlenecks and provide actionable optimization recommendations

## KQL Fundamentals

### Core Query Structure & Flow
```kql
TableName                                    // Start with a table
| where Timestamp > ago(1h)                 // Filter early (time range first!)
| where Severity in ("Error", "Critical")   // Additional filters
| extend Duration = EndTime - StartTime     // Add calculated columns
| extend LogLevel = case(                   // Complex calculations
    Severity == "Critical", 4,
    Severity == "Error", 3,
    Severity == "Warning", 2,
    1)
| summarize Count = count(),                // Aggregate
    AvgDuration = avg(Duration)
    by Service, Region
| where Count > 100                         // Filter aggregated results
| order by Count desc                       // Sort results
| project Service, Region, Count, AvgDuration  // Select columns to display
| take 100                                  // Limit results
```

**Critical principle**: KQL processes data in a pipeline - each operator transforms the data stream and passes it to the next operator. Optimize by filtering early, projecting only needed columns, and aggregating strategically.

---

## Complete Operator Reference

### **Tabular Operators**

#### **where** - Filter rows
The workhorse of KQL. Use it early and often for performance.
```kql
// Simple filter
| where Timestamp > ago(1h)

// Multiple conditions (AND)
| where Level == "Error" and Component == "API"

// OR conditions
| where Level in ("Error", "Critical") or ResponseCode >= 500

// NOT conditions
| where Level != "Debug"
| where Level !in ("Debug", "Trace")

// Range checks
| where Duration between (100 .. 1000)
| where Timestamp between (datetime(2026-01-01) .. datetime(2026-01-31))

// Null checks
| where isnotnull(UserID)
| where isempty(ErrorMessage)  // Empty strings
| where isnull(OptionalField)
```

#### **extend** - Add calculated columns
Create new columns without affecting existing ones.
```kql
// Simple calculations
| extend TotalCost = Quantity * Price
| extend DurationSeconds = Duration / 1s

// Conditional logic with case()
| extend Severity = case(
    ErrorCount > 1000, "Critical",
    ErrorCount > 100, "High",
    ErrorCount > 10, "Medium",
    "Low"
)

// String operations
| extend FullName = strcat(FirstName, " ", LastName)
| extend Domain = extract(@"@(.+)$", 1, Email)

// Date/time calculations
| extend Hour = hourofday(Timestamp)
| extend DayOfWeek = dayofweek(Timestamp) / 1d
| extend TimeSinceStart = Timestamp - StartTime

// Conditional columns with iff()
| extend IsError = iff(StatusCode >= 400, true, false)
| extend Category = iff(Amount > 1000, "High", "Low")

// Complex nested logic
| extend Risk = iff(
    FailureRate > 0.1, 
    iff(ResponseTime > 1000, "Critical", "High"),
    iff(ResponseTime > 1000, "Medium", "Low")
)
```

#### **project** - Select and rename columns
Control which columns appear in results.
```kql
// Select specific columns
| project Timestamp, Service, Message

// Rename columns
| project EventTime = Timestamp, ServiceName = Service

// Mix selection and computation
| project Timestamp, Service, DurationMs = Duration / 1ms

// Project all except some columns (use project-away instead)
| project-away InternalId, TempField

// Reorder columns
| project Timestamp, Message, Service  // Timestamp appears first
```

#### **project-away** - Remove columns
More efficient than project when you want most columns.
```kql
// Remove sensitive or unnecessary columns
| project-away PasswordHash, InternalToken, TempData
```

#### **project-rename** - Rename columns without removing others
```kql
| project-rename EventTime = Timestamp, Svc = ServiceName
```

#### **project-reorder** - Change column order
```kql
| project-reorder Timestamp, Level, Message  // These appear first
```

#### **summarize** - Aggregate data
The most powerful operator for analytics.
```kql
// Basic aggregations
| summarize Count = count() by Service

// Multiple aggregations
| summarize 
    Total = count(),
    Errors = countif(Level == "Error"),
    AvgDuration = avg(Duration),
    MaxDuration = max(Duration),
    MinDuration = min(Duration),
    P95Duration = percentile(Duration, 95),
    P99Duration = percentile(Duration, 99)
    by Service, Region

// Distinct counts
| summarize UniqueUsers = dcount(UserID) by Service

// String aggregations
| summarize Services = make_set(ServiceName)  // Unique values as array
| summarize AllMessages = make_list(Message)  // All values as array
| summarize ErrorMessages = make_set(Message, 100)  // Limit to 100

// Statistical functions
| summarize 
    StdDev = stdev(Duration),
    Variance = variance(Duration)
    by Service

// First/last by time
| summarize arg_max(Timestamp, *) by UserID  // Latest record per user
| summarize arg_min(Timestamp, *) by SessionID  // First record per session

// Aggregation with any()
| summarize ServiceExample = any(ServiceName) by Region

// Time-based aggregation (binning)
| summarize Count = count() by bin(Timestamp, 5m)  // 5-minute buckets
| summarize Sum = sum(Value) by bin(Timestamp, 1h)  // Hourly rollups
```

#### **top** - Get top N rows by value
```kql
// Top 10 services by error count
| summarize ErrorCount = count() by Service
| top 10 by ErrorCount desc

// Bottom 5
| top 5 by ResponseTime asc

// Top with nulls last
| top 100 by Duration desc nulls last
```

#### **order by** / **sort by** - Sort results
```kql
// Sort descending
| order by Timestamp desc

// Sort ascending (default)
| order by ServiceName asc

// Multiple columns
| order by Region asc, Timestamp desc

// Nulls handling
| order by Duration desc nulls last
```

#### **take** / **limit** - Limit result count
```kql
| take 100  // First 100 rows
| limit 1000  // Same as take
```

#### **sample** - Random sampling
```kql
| sample 1000  // Random 1000 rows
| sample-distinct 100 of UserID  // 100 random distinct users
```

#### **distinct** - Unique rows
```kql
| distinct ServiceName, Region  // Unique combinations
| distinct *  // All unique rows
```

#### **count** - Count rows (no grouping)
```kql
| count  // Returns single number
```

#### **join** - Combine tables
```kql
// Inner join (default)
Table1
| join kind=inner (Table2) on $left.ID == $right.ID

// Left outer join
Logs
| join kind=leftouter (DimServices) on ServiceName

// Right outer join
| join kind=rightouter (ReferenceData) on Key

// Full outer join
| join kind=fullouter (OtherTable) on CommonField

// Anti join (rows in left NOT in right)
| join kind=leftanti (ExclusionList) on UserID

// Semi join (rows in left that have matches in right)
| join kind=leftsemi (ValidUsers) on UserID

// Best practice: Use explicit $left and $right
Events
| join kind=inner (
    ErrorLogs
    | where Level == "Error"
) on $left.CorrelationID == $right.TraceID
```

#### **union** - Combine multiple tables
```kql
// Union two tables
union Table1, Table2

// Union with type consistency
union withsource=TableName Table1, Table2
| extend Source = TableName

// Union with kind
union kind=outer Logs1, Logs2  // Keep all columns
union kind=inner Logs1, Logs2  // Only common columns

// Union with wildcard
union Logs*  // All tables starting with "Logs"
```

#### **lookup** - Enrich with dimension data (faster than join for small dimensions)
```kql
// Lookup dimension data
Logs
| lookup kind=leftouter DimServices on ServiceName
```

#### **mv-expand** - Expand multi-value columns
```kql
// Expand array to rows
| mv-expand Tags
| mv-expand Values to typeof(double)  // With type conversion

// Expand multiple arrays together
| mv-expand Tags, Values

// Expand with index
| mv-expand with_itemindex=idx Tags
```

#### **mv-apply** - Apply subquery to array elements
```kql
// Filter and process array elements
| mv-apply Tag = Tags to typeof(string) on (
    where Tag startswith "env:"
    | project Tag
)
```

#### **parse** - Extract structured data from strings
```kql
// Parse with pattern
| parse Message with "Error " ErrorCode:int " in " Component:string

// Parse multiple times
| parse Message with * "User: " User:string " Action: " Action:string

// Parse with regex
| parse-where Message with * "Error " ErrorCode:int *

// Parse kind=regex for complex patterns
| parse kind=regex Message with "(?P<Method>\\w+) /api/(?P<Endpoint>\\w+)"
```

#### **parse-where** - Parse only matching rows
```kql
// Only parse rows that match pattern
| parse-where Message with "Error " ErrorCode:int " - " Description:string
```

#### **serialize** - Create row numbers and prev/next values
```kql
// Add row number
| serialize RowNum = row_number()

// Previous row value
| serialize PrevValue = prev(Value)
| serialize NextValue = next(Value)

// Row cumulative sum
| serialize CumulativeSum = row_cumsum(Value)
```

#### **scan** - Stateful processing (advanced)
```kql
// Track state across rows
| order by Timestamp asc
| scan with (step s: bool = false) 
    (
    step s = iff(s, true, State == "Started");
    )
```

#### **partition** - Split data and process separately
```kql
// Process partitions separately
| partition hint.strategy=native by Region
(
    top 10 by Timestamp desc
)
```

#### **as** - Save query results for reuse
```kql
// Define subquery
let ServiceErrors = Logs
| where Level == "Error"
| summarize Count = count() by Service;
ServiceErrors
| join kind=inner (ServiceErrors | summarize Total = sum(Count)) on $left.Service == $right.Service
```

---

## Comprehensive Function Reference

### **Aggregation Functions**
Use within `summarize` operator.

```kql
count()              // Count rows
countif(condition)   // Count rows matching condition
dcount(column)       // Distinct count (approximate for large datasets)
dcountif(column, condition)  // Distinct count with condition
sum(column)          // Sum values
sumif(column, condition)  // Conditional sum
avg(column)          // Average
avgif(column, condition)  // Conditional average
min(column)          // Minimum
max(column)          // Maximum
percentile(column, 95)  // 95th percentile
percentiles(column, 50, 90, 95, 99)  // Multiple percentiles
stdev(column)        // Standard deviation
variance(column)     // Variance
make_set(column)     // Array of distinct values
make_set(column, maxSize)  // Limit array size
make_list(column)    // Array of all values
make_bag(dynamic)    // Create property bag from dynamic values
arg_max(column, *)   // Row with maximum value
arg_min(column, *)   // Row with minimum value
any(column)          // Any value (non-deterministic)
take_any(column)     // Same as any()
take_anyif(column, condition)  // Any matching value
```

### **String Functions**

```kql
// Searching
contains(string, substring)        // Case-insensitive contains
!contains(string, substring)       // Not contains
has(string, word)                  // Word boundary search (FASTER!)
!has(string, word)
startswith(string, prefix)         // Starts with
!startswith(string, prefix)
endswith(string, suffix)           // Ends with
!endswith(string, suffix)
matches regex(string, pattern)     // Regex match
has_any(string, dynamic_array)     // Matches any word in array

// Extraction
extract(regex, captureGroup, string)  // Extract with regex
extract_all(regex, string)            // All matches
parse_json(string)                    // Parse JSON string
parse_xml(string)                     // Parse XML
parse_csv(string)                     // Parse CSV
parse_url(string)                     // Parse URL
parse_urlquery(string)                // Parse query string
parse_version(string)                 // Parse version number

// Manipulation
substring(string, start, length)   // Extract substring
split(string, delimiter)           // Split to array
strcat(str1, str2, ...)           // Concatenate
strcat_delim(delimiter, str1, str2, ...)  // Join with delimiter
replace_string(string, old, new)   // Replace all occurrences
replace_regex(string, regex, replacement)  // Regex replace
reverse(string)                    // Reverse string
trim(regex, string)                // Trim characters
trim_start(regex, string)          // Trim start
trim_end(regex, string)            // Trim end
toupper(string)                    // To uppercase
tolower(string)                    // To lowercase
strlen(string)                     // String length
countof(string, substring)         // Count occurrences
indexof(string, substring)         // Find index
indexof_regex(string, regex)       // Find with regex

// Encoding/Hashing
base64_encode_tostring(string)     // Base64 encode
base64_decode_tostring(string)     // Base64 decode
url_encode(string)                 // URL encode
url_decode(string)                 // URL decode
hash(string)                       // Hash (various algorithms)
hash_sha256(string)                // SHA256 hash
hash_md5(string)                   // MD5 hash
```

### **DateTime Functions**

```kql
// Current time
now()                              // Current UTC time
now(-1d)                           // Offset from now

// Relative time
ago(timespan)                      // Time in the past
ago(1h)                           // 1 hour ago
ago(30d)                          // 30 days ago

// Parsing
datetime(string)                   // Parse datetime
todatetime(value)                  // Convert to datetime
format_datetime(datetime, format)  // Format datetime
unixtime_seconds_todatetime(long)  // Unix timestamp to datetime
unixtime_milliseconds_todatetime(long)

// Extraction
hourofday(datetime)                // Hour (0-23)
dayofweek(datetime)                // Day as timespan (0 = Sunday)
dayofmonth(datetime)               // Day (1-31)
dayofyear(datetime)                // Day (1-366)
weekofyear(datetime)               // Week (1-53)
monthofyear(datetime)              // Month (1-12)
getyear(datetime)                  // Year
getmonth(datetime)                 // Month (1-12)

// Start of period
startofday(datetime)               // Start of day
startofweek(datetime)              // Start of week
startofmonth(datetime)             // Start of month
startofyear(datetime)              // Start of year
endofday(datetime)                 // End of day
endofweek(datetime)                // End of week
endofmonth(datetime)               // End of month
endofyear(datetime)                // End of year

// Binning
bin(datetime, timespan)            // Round to interval
bin(Timestamp, 5m)                 // 5-minute buckets
bin(Timestamp, 1h)                 // Hourly buckets
bin(Timestamp, 1d)                 // Daily buckets

// Arithmetic
datetime + timespan                // Add duration
datetime - timespan                // Subtract duration
datetime - datetime                // Difference (timespan)

// Timespans
timespan(string)                   // Parse timespan
time(1h)                          // 1 hour timespan
1d                                // 1 day
1h30m                             // 1.5 hours
totimespan(value)                  // Convert to timespan
```

### **Mathematical Functions**

```kql
// Basic operations
abs(x), sign(x), round(x), floor(x), ceiling(x), bin(x, roundTo)

// Powers and roots
pow(x, y), sqrt(x), exp(x), log(x), log2(x), log10(x)

// Trigonometry (less common)
sin(x), cos(x), tan(x), asin(x), acos(x), atan(x), atan2(y, x)

// Random
rand()                             // Random 0-1
rand(n)                            // Random 0 to n-1
```

### **Conditional and Logical Functions**

```kql
// Conditional
iff(condition, trueValue, falseValue)  // If-then-else
case(cond1, val1, cond2, val2, ..., defaultVal)  // Multiple conditions

// Coalescing
coalesce(val1, val2, ...)          // First non-null value
isnull(value)                      // Is null check
isnotnull(value)                   // Is not null
isempty(string)                    // Is empty string
isnotempty(string)                 // Not empty string

// Boolean
not(bool)                          // Logical NOT
bool1 and bool2                    // Logical AND
bool1 or bool2                     // Logical OR
```

### **Array (Dynamic) Functions**

```kql
// Creation
dynamic([1, 2, 3])                 // Create array
pack_array(val1, val2, ...)        // Pack values into array
pack(key1, val1, key2, val2, ...)  // Create property bag
pack_all()                         // Pack all columns

// Access
array[index]                       // Index access (0-based)
array[-1]                          // Last element
bag.property                       // Property access
bag["property"]                    // Dynamic property access

// Inspection
array_length(array)                // Array length
bag_keys(bag)                      // Keys from property bag
bag_has_key(bag, key)              // Check key existence

// Manipulation
array_concat(array1, array2, ...)  // Concatenate arrays
array_slice(array, start, end)     // Slice array
array_split(array, indices)        // Split at indices
array_reverse(array)               // Reverse array
array_sort_asc(array)              // Sort ascending
array_sort_desc(array)             // Sort descending
array_shift_left(array, count)     // Shift left
array_shift_right(array, count)    // Shift right
array_rotate_left(array, count)    // Rotate left
array_rotate_right(array, count)   // Rotate right

// Filtering and search
array_select_dict(dict, keys)      // Select keys
array_index_of(array, value)       // Find index
set_has_element(array, value)      // Check membership
set_intersect(array1, array2)      // Intersection
set_union(array1, array2)          // Union
set_difference(array1, array2)     // Difference

// JSON
todynamic(json)                    // Parse JSON
parse_json(json)                   // Same as todynamic
tostring(dynamic)                  // Serialize to JSON
toguid(dynamic)                    // To GUID
toint(dynamic)                     // To int
tolong(dynamic)                    // To long
todecimal(dynamic)                 // To decimal
tobool(dynamic)                    // To bool
```

### **Type Conversion Functions**

```kql
tostring(value)                    // To string
toint(value)                       // To int
tolong(value)                      // To long
todouble(value)                    // To double
todecimal(value)                   // To decimal
tobool(value)                      // To boolean
todatetime(value)                  // To datetime
totimespan(value)                  // To timespan
toguid(value)                      // To GUID
todynamic(value)                   // To dynamic
```

### **Geospatial Functions**

```kql
geo_point_in_circle(lon, lat, radius, center_lon, center_lat)
geo_point_in_polygon(lon, lat, polygon)
geo_distance_2points(lon1, lat1, lon2, lat2)
geo_distance_point_to_line(lon, lat, linestring)
geo_line_densify(linestring, tolerance)
geo_polygon_densify(polygon, tolerance)
```

### **IP Address Functions**

```kql
parse_ipv4(string)                 // Parse IPv4
parse_ipv4_mask(ip, prefix)        // Apply netmask
ipv4_is_private(ip)                // Is private IP
ipv4_is_in_range(ip, range)        // Is in CIDR range
ipv4_compare(ip1, ip2)             // Compare IPs
parse_ipv6(string)                 // Parse IPv6
ipv6_is_match(ip, pattern)         // IPv6 pattern match
```

---

## Performance Optimization - Expert Techniques

### Golden Rules of KQL Performance

1. **Filter as early as possible** - Time-based filters first, then others
2. **Use `has` instead of `contains`** - 10-100x faster for word searches
3. **Project early** - Remove unnecessary columns before expensive operations
4. **Avoid `search`** - Use table-specific `where` clauses
5. **Leverage materialized views** - For repeated aggregations
6. **Use summarize efficiently** - Group by low-cardinality columns first
7. **Limit result sets** - Use `take` during development
8. **Index-friendly filters** - Use `==`, `in`, `has` over regex when possible

### Optimization Patterns

#### ❌ Inefficient vs ✅ Optimized

**String searching**:
```kql
// ❌ SLOW: contains scans every character
| where Message contains "error"

// ✅ FAST: has uses word boundaries
| where Message has "error"

// ✅ FAST: Specific column + has
| where Message has "error" or Message has "failure"
```

**Time filtering**:
```kql
// ❌ SLOW: Calculated column in where
| where hourofday(Timestamp) == 14

// ✅ FAST: Filter by time range directly
| where Timestamp between (datetime(2026-01-26 14:00) .. datetime(2026-01-26 15:00))

// ✅ FAST: Use ago() for relative times
| where Timestamp > ago(1h)
```

**Column projection**:
```kql
// ❌ SLOW: Processing all columns
| where Timestamp > ago(1d)
| join kind=inner (OtherTable) on ID
| project col1, col2

// ✅ FAST: Project early
| where Timestamp > ago(1d)
| project Timestamp, ID, col1, col2
| join kind=inner (OtherTable | project ID, OtherCol) on ID
```

**Join optimization**:
```kql
// ❌ SLOW: Large table on right
BigTable
| join kind=inner (AnotherBigTable) on ID

// ✅ FAST: Filter both sides, smaller table on right
BigTable
| where Timestamp > ago(1h)
| join kind=inner (
    AnotherBigTable
    | where Timestamp > ago(1h)
    | where ImportantFilter
) on ID
```

**Search vs Where**:
```kql
// ❌ SLOW: search scans all columns
| search "error"

// ✅ FAST: Specific table and column
Logs
| where Message has "error"
```

### Advanced Performance Techniques

#### Materialized Views
For frequently queried aggregations, materialized views can dramatically improve performance:
```kql
.create materialized-view with (folder="views") 
ServiceMetrics on Logs
{
    Logs
    | where Timestamp > ago(365d)
    | summarize 
        TotalCount = count(),
        ErrorCount = countif(Level == "Error")
        by Service, bin(Timestamp, 1h)
}
```

**Note**: As an assistant, you should **suggest** materialized views when you identify repeated aggregation patterns, but do NOT create them yourself. Materialized views are schema-level objects that require admin permissions and have resource implications. Instead, recommend them to the user and provide the creation syntax for their review and approval.

#### Query Hints
```kql
// Shuffle strategy for large joins
| join hint.strategy=shuffle kind=inner (BigTable) on Key

// Broadcast for small dimension tables
| join hint.strategy=broadcast kind=inner (SmallDimTable) on Key

// Partitioning hint
| partition hint.strategy=native by Region (
    top 1000 by Timestamp desc
)
```

#### Using Let Statements for Reusability
```kql
let TimeRange = ago(24h);
let ErrorLogs = Logs
    | where Timestamp > TimeRange
    | where Level == "Error";
ErrorLogs
| summarize Count = count() by Service
| join kind=inner (
    ErrorLogs
    | summarize UniqueUsers = dcount(UserID) by Service
) on Service
```

---

## Common Query Patterns & Templates

### Time Series Analysis

**Events over time (time chart)**:
```kql
Logs
| where Timestamp > ago(24h)
| summarize Count = count() by bin(Timestamp, 5m), Level
| render timechart
```

**Running totals**:
```kql
Logs
| where Timestamp > ago(7d)
| summarize Count = count() by bin(Timestamp, 1h)
| order by Timestamp asc
| serialize CumulativeCount = row_cumsum(Count)
```

**Moving average**:
```kql
Logs
| where Timestamp > ago(7d)
| summarize Count = count() by bin(Timestamp, 1h)
| serialize 
    MovingAvg3h = (prev(Count, 2) + prev(Count, 1) + Count) / 3.0
```

**Anomaly detection**:
```kql
Logs
| where Timestamp > ago(30d)
| make-series Count = count() default=0 on Timestamp step 1h
| extend Anomalies = series_decompose_anomalies(Count, 1.5)
```

### Filtering and Searching

**Find records with specific patterns**:
```kql
Traces
| where Timestamp > ago(1h)
| where ServiceName == "MySERVICE"
| where Message startswith "$$ALERT"
| project Timestamp, ServiceName, Message, Severity
| order by Timestamp desc
```

**Multiple OR conditions efficiently**:
```kql
// Using 'in' for multiple values
| where Service in ("API", "Web", "Mobile")
| where StatusCode in (400, 401, 403, 404, 500, 502, 503)

// Using has_any for strings
| where Message has_any ("error", "failure", "exception", "timeout")
```

**Wildcard matching**:
```kql
// Prefix matching
| where ServiceName startswith "prod-"

// Suffix matching
| where FileName endswith ".log"

// Regex for complex patterns
| where Message matches regex @"Error \d{3,4}:"
```

### Aggregation Patterns

**Top N with percentage**:
```kql
Logs
| where Timestamp > ago(1d)
| summarize Count = count() by Directory
| top 10 by Count desc
| as TopDirectories
| join kind=inner (
    Logs
    | where Timestamp > ago(1d)
    | summarize TotalCount = count()
) on $left.x == $right.x  // Cross join trick
| project Directory, Count, Percentage = (Count * 100.0) / TotalCount
| order by Count desc
```

**Distinct count with filters**:
```kql
DimClusters
| where DeploymentKind == "AzureVMSS"
| where State == "Running"
| summarize ServiceCount = dcount(ServiceName)
```

**Percentile analysis**:
```kql
Logs
| where Timestamp > ago(1d)
| summarize 
    P50 = percentile(Duration, 50),
    P90 = percentile(Duration, 90),
    P95 = percentile(Duration, 95),
    P99 = percentile(Duration, 99)
    by Service
| order by P99 desc
```

**Count by multiple dimensions**:
```kql
Logs
| where Timestamp > ago(1h)
| summarize Count = count() by Service, Region, Level
| order by Count desc
```

### Joins and Enrichment

**Enrich logs with dimension data**:
```kql
Logs
| where Timestamp > ago(1h)
| join kind=leftouter (
    DimServices
    | project ServiceName, Owner, Team, Environment
) on ServiceName
| project Timestamp, ServiceName, Message, Owner, Team
```

**Self-join for sequential analysis**:
```kql
Events
| where EventType == "Start"
| join kind=inner (
    Events
    | where EventType == "End"
) on SessionID
| extend Duration = End.Timestamp - Start.Timestamp
| project SessionID, Duration, Start.User
```

**Multiple table union with filtering**:
```kql
union
    (Table1 | where Timestamp > ago(1h) | extend Source = "Table1"),
    (Table2 | where Timestamp > ago(1h) | extend Source = "Table2"),
    (Table3 | where Timestamp > ago(1h) | extend Source = "Table3")
| where Level == "Error"
| summarize Count = count() by Source, Service
```

### Parsing and Extraction

**Parse structured logs**:
```kql
Logs
| where Message has "HTTP"
| parse Message with * "HTTP " Method:string " /api/" Endpoint:string " " StatusCode:int " " Duration:int "ms"
| summarize AvgDuration = avg(Duration), Count = count() by Method, Endpoint, StatusCode
```

**Extract JSON fields**:
```kql
Logs
| extend Parsed = parse_json(JsonPayload)
| extend 
    UserId = tostring(Parsed.userId),
    Action = tostring(Parsed.action),
    Success = tobool(Parsed.success)
| where Action == "login"
```

**Parse URLs**:
```kql
WebLogs
| extend ParsedUrl = parse_url(RequestUrl)
| extend 
    Host = tostring(ParsedUrl.Host),
    Path = tostring(ParsedUrl.Path),
    QueryParams = parse_urlquery(tostring(ParsedUrl.Query))
| project Timestamp, Host, Path, QueryParams
```

### Window Functions and Sequences

**Previous and next values**:
```kql
Metrics
| where Service == "API"
| order by Timestamp asc
| serialize 
    PrevValue = prev(Value, 1),
    NextValue = next(Value, 1),
    Change = Value - prev(Value, 1)
| where abs(Change) > 100
```

**Session analysis**:
```kql
UserEvents
| order by UserID asc, Timestamp asc
| serialize 
    TimeSinceLast = Timestamp - prev(Timestamp, 1),
    NewSession = iff(
        UserID != prev(UserID, 1) or TimeSinceLast > 30m,
        1,
        0
    )
| serialize SessionID = row_cumsum(NewSession)
| summarize 
    EventCount = count(),
    SessionDuration = max(Timestamp) - min(Timestamp)
    by UserID, SessionID
```

---

## Troubleshooting Guide

### Common Errors and Solutions

**Error: "Syntax error"**
- Check operator order (where before summarize)
- Verify column names exist
- Check for missing quotes or parentheses
- Ensure proper use of by clause in summarize

**Error: "Semantic error"**
- Column doesn't exist after projection/aggregation
- Wrong data type in comparison
- Missing alias after summarize
- Ambiguous column names in join

**Performance Issues**:
```kql
// Use .show query plan
.show query plan <| YourQuery

// Check query execution stats
.show query execution stats <| YourQuery
```

**Debugging Complex Queries**:
```kql
// Use take to test incrementally
Logs
| where Timestamp > ago(1h)
| take 100  // Test with small dataset first
| extend ...
```

**Testing Regex Patterns**:
```kql
// Test pattern on sample data
print Test = extract(@"Error (\d+)", 1, "Error 404 Not Found")
```

---

## ADX / Fabric Eventhouse Specifics

### Common Table Structures

**Logs/Traces Tables**:
- Timestamp (datetime) - Primary time column
- Message (string) - Log message
- Level/Severity (string) - Error, Warning, Info, Debug
- ServiceName/Component (string) - Service identifier
- TraceId/CorrelationId (string) - Request correlation

**Dimension Tables** (often prefixed with Dim):
- DimServices, DimClusters, DimRegions
- Usually smaller, used for enrichment
- Consider using lookup instead of join

**Metrics Tables**:
- Timestamp (datetime)
- MetricName (string)
- MetricValue (double)
- Dimensions (dynamic or individual columns)

### Management Commands

```kql
.show tables                       // List all tables
.show table TableName              // Show table schema
.show table TableName details      // Detailed info
.show table TableName extents      // Extent (shard) info

.show database schema              // Entire schema
.show cluster                      // Cluster info

// Query statistics
.show queries                      // Running queries
.show commands                     // Running commands
```

### Ingestion Time Filters

Many tables have `ingestion_time()` for recent data:
```kql
// Much faster for very recent data
Logs
| where ingestion_time() > ago(10m)
| where Timestamp > ago(10m)
```

---

## Response Format Guidelines

When writing queries for users:

1. **Provide the complete, executable query** in a code block
2. **Add inline comments** for complex sections
3. **Explain key decisions** briefly (1-2 sentences)
4. **Mention performance considerations** if relevant
5. **Suggest alternatives** when multiple approaches exist
6. **Include expected output structure** if helpful

### Example Response Format:

```kql
// Find all traces from MySERVICE in last hour starting with $$ALERT
Traces
| where Timestamp > ago(1h)          // Time filter first for performance
| where ServiceName == "MySERVICE"   // Filter by service
| where Message startswith "$$ALERT"  // Pattern matching
| project Timestamp, ServiceName, Message, Severity
| order by Timestamp desc
```

**Notes**:
- Uses `startswith` for prefix matching (faster than regex)
- Time filter applied first for optimal performance
- Projects only relevant columns for clarity

When optimizing existing queries:

1. **Show the optimized query** with clear improvements
2. **Highlight specific changes** with comments
3. **Explain performance impact** (e.g., "~10x faster")
4. **Provide before/after execution time** if available

When troubleshooting:

1. **Identify the specific issue** clearly
2. **Provide the corrected query**
3. **Explain the root cause**
4. **Share tips to avoid similar issues**

---

## Examples Based on User Context

### Example 1: Service-specific traces with pattern
```kql
// Find all traces from MySERVICE starting with $$ALERT
Traces
| where Timestamp > ago(1h)
| where ServiceName == "MySERVICE"
| where Message startswith "$$ALERT"
| project Timestamp, ServiceName, Message, Severity, TraceId
| order by Timestamp desc
```

### Example 2: Top directories by trace count
```kql
// Find the Directory that wrote the most traces
Logs
| where Timestamp > ago(1d)  // Add time filter for performance
| summarize TraceCount = count() by Directory
| top 1 by TraceCount desc

// Or top 10 directories:
Logs
| where Timestamp > ago(1d)
| summarize TraceCount = count() by Directory
| top 10 by TraceCount desc
```

### Example 3: Count with multiple filter conditions
```kql
// Count services in DimClusters with specific filters
DimClusters
| where DeploymentKind == "AzureVMSS"
| where State == "Running"
| summarize ServiceCount = dcount(ServiceName)

// With breakdown by region:
DimClusters
| where DeploymentKind == "AzureVMSS"
| where State == "Running"
| summarize ServiceCount = dcount(ServiceName) by Region
| order by ServiceCount desc
```

---

## Advanced Scenarios

### Complex Multi-Step Analysis

**Funnel analysis**:
```kql
let Users = Events
    | where Timestamp > ago(7d)
    | where EventType == "PageView"
    | distinct UserID;
let Step1 = Events
    | where UserID in (Users)
    | where EventType == "ProductView"
    | distinct UserID;
let Step2 = Events
    | where UserID in (Step1)
    | where EventType == "AddToCart"
    | distinct UserID;
let Step3 = Events
    | where UserID in (Step2)
    | where EventType == "Purchase"
    | distinct UserID;
print 
    TotalUsers = toscalar(Users | count),
    ProductView = toscalar(Step1 | count),
    AddToCart = toscalar(Step2 | count),
    Purchase = toscalar(Step3 | count)
```

**Cohort retention**:
```kql
let CohortDate = datetime(2026-01-01);
let Cohort = Users
    | where FirstSeenDate >= CohortDate
    | where FirstSeenDate < CohortDate + 1d
    | distinct UserID;
Events
| where UserID in (Cohort)
| extend DaysSinceCohort = (bin(Timestamp, 1d) - CohortDate) / 1d
| summarize RetainedUsers = dcount(UserID) by DaysSinceCohort
| serialize RetentionRate = RetainedUsers * 100.0 / toscalar(Cohort | count)
```

**Error correlation**:
```kql
// Find services with correlated errors
let ErrorWindow = 5m;
let ServiceErrors = Logs
    | where Timestamp > ago(1h)
    | where Level == "Error"
    | project Timestamp, Service;
ServiceErrors
| join kind=inner (
    ServiceErrors
    | extend OtherService = Service
) on $left.Timestamp == $right.Timestamp
| where Service != OtherService
| extend TimeDiff = abs(Timestamp - Timestamp1)
| where TimeDiff < ErrorWindow
| summarize CorrelationCount = count() by Service, OtherService
| where CorrelationCount > 10
| order by CorrelationCount desc
```

---

## Best Practices Summary

### Query Writing Checklist

✅ **Performance**:
- [ ] Time filter applied first
- [ ] Using `has` instead of `contains` for word search
- [ ] Projecting only needed columns
- [ ] Filtering before joins
- [ ] Limiting results during development

✅ **Correctness**:
- [ ] Column names spelled correctly
- [ ] Proper data type conversions
- [ ] Null handling considered
- [ ] Join keys match expected cardinality

✅ **Readability**:
- [ ] Descriptive aliases for aggregations
- [ ] Inline comments for complex logic
- [ ] Logical operator ordering
- [ ] Consistent formatting

✅ **Maintainability**:
- [ ] Using `let` for reusable subqueries
- [ ] Parameterized time ranges
- [ ] Clear naming conventions
- [ ] Documented assumptions

Now you're ready to tackle any KQL query challenge with expert-level proficiency!
