
# MongoDB Index Optimization Report

## Summary

| Collection | Status | Issue | Action |
|---|---|---|---|
| FileVersions | 🔴 Critical | COLLSCAN — 157K docs examined | Add compound index |
| Files | 🔴 Critical | IXSCAN but 10,982 docs examined for 100 returned | Review query & index coverage |
| CoscompStockInfos | 🔴 Critical | 917ms for 450 docs | Needs index investigation |
| Movres | 🟡 Monitor | Only 318 docs now — acceptable | Re-evaluate when volume grows |
| Users | 🟢 OK | Using IXSCAN mostly | No action needed |
| StockChartHistoricalPrices | 🟢 OK | Index already added | — |
| UserReadableDatas | 🟢 OK | Only 13 documents | No index needed |
| UserRoleMaps | ⚪ N/A | Not examined | — |
| UserProfiles | ⚪ N/A | Not examined | — |
| UserActivationKeyMaps | ⚪ N/A | Not examined | — |

---

## 🔴 FileVersions — Critical

### Current State

| Metric | Before | After Index |
|---|---|---|
| Plan | `COLLSCAN` | `IXSCAN` |
| `docsExamined` | ~157,000 | ~1 |
| `millis` | ~3,366ms | < 5ms |

### Fix

```js
db.FileVersions.createIndex({ FileId: 1, No: -1 })
```

### Why

Queries on `FileVersions` filter by `FileId` and sort/query by `No` descending.
Without an index, MongoDB scans all ~157K documents. The compound index allows it to jump directly to matching documents.

---

## 🔴 Files — Critical

### Current State

| Metric | Value |
|---|---|
| Plan | `IXSCAN` (index exists) |
| `docsExamined` | 10,982 |
| `nreturned` | 100 |
| `millis` | ~3,400ms |

### Problem

An index is being used, but the ratio of `docsExamined` to `nreturned` is **110:1** — meaning the index is filtering to a large candidate set, then doing in-memory work to return only 100 docs. Performance will degrade linearly as document count grows.

### Possible Causes

- Index does not cover all query fields (missing filter fields in the index)
- Query uses range operators (`$gt`, `$lt`, `$regex`) that limit index efficiency
- Sort field is not included in the index — causing a blocking sort after index scan
- Index field order doesn't match query selectivity (most selective field should be first)

### Recommended Actions

1. Run `explain("executionStats")` on the slow query and check `executionStages` in detail
2. Ensure sort field is the **last** field in the index
3. Consider a **covered index** — include all projected fields so no document fetch is needed
4. If a regex is used, consider restructuring to a prefix match (`/^value/`) or a dedicated search field

---

## 🔴 CoscompStockInfos — Critical

### Current State

| Metric | Value |
|---|---|
| Query | `find` |
| `docsExamined` | 450 |
| `nreturned` | 1 |
| `millis` | ~917ms |

### Problem

Examining 450 documents to return 1 result at ~917ms is extremely slow — indicates either:
- No index on the query field (COLLSCAN on a small collection)
- Index exists but the field has low cardinality or wrong type
- The 917ms may include network/lock wait time, not just scan time

### Recommended Actions

1. Run `explain("executionStats")` to confirm `COLLSCAN` vs `IXSCAN`
2. Identify the query filter field and add a targeted index:

```js
// Example — replace `queryField` with the actual field name
db.CoscompStockInfos.createIndex({ queryField: 1 })
```

3. If the collection holds stock data queried by ticker/symbol + date:

```js
db.CoscompStockInfos.createIndex({ symbol: 1, date: -1 })
```

4. Check for lock contention or missing index hint using `db.currentOp()`

---

## 🟡 Movres — Monitor

### Current State

| Metric | Value |
|---|---|
| Document count | ~318 |
| Status | Acceptable for now |

### Notes

At 318 documents, full collection scans are fast. However, if the collection grows significantly, slow queries will emerge. Pre-emptively add an index once you identify the primary query pattern.

---

## General Recommendations

### Diagnosing Slow Queries

```js
// Get full execution stats for any query
db.CollectionName.find({ field: value }).explain("executionStats")

// Key fields to check in output:
// executionStats.totalDocsExamined  — docs scanned
// executionStats.nReturned          — docs returned
// executionStats.executionTimeMillis
// executionStages.stage             — COLLSCAN is bad, IXSCAN is good
```

### Index Design Rules

| Rule | Detail |
|---|---|
| **ESR Rule** | Order index fields: Equality → Sort → Range |
| **Covered queries** | Include all projected fields to avoid document fetch |
| **Compound over single** | A compound index on `{ a, b }` covers queries on `a` alone too |
| **Avoid over-indexing** | Each index adds write overhead — only index what is queried |

### Monitoring Going Forward

```js
// Find collections with slow queries (requires profiling enabled)
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find().sort({ millis: -1 }).limit(10)

// Check index usage stats
db.CollectionName.aggregate([{ $indexStats: {} }])
```
