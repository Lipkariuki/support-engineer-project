# Production Investigation 01 — Reading a PostgreSQL Execution Plan

> Learning how to investigate a real execution plan like a PostgreSQL Support Engineer.

---

# Scenario

A customer reports:

> "This query is taking longer than expected."

Instead of guessing, we collect evidence by running:

```sql
EXPLAIN ANALYZE
```

The goal is not to immediately optimize the query.

The goal is to understand **why PostgreSQL chose a particular execution plan.**

---

# Step 1 — Are Indexes Being Used?

The execution plan shows multiple Index Scans.

Examples:

```
Index Scan Backward using job_shifts_to_date_time_ts

Index Scan using nursa_job_job_id_idx

Index Scan using idx_job_id

Index Scan using nursa_user_pkey
```

## Observation

PostgreSQL is already using several indexes.

This tells us that the planner considers these indexes beneficial for parts of the query.

## Lesson

Finding Index Scans in an execution plan is one of the first things a Support Engineer looks for.

---

# Step 2 — Is PostgreSQL Performing a Sequential Scan?

The execution plan contains:

```
Parallel Seq Scan on shifts_timekeeping
```

## Observation

A Sequential Scan does not automatically mean something is wrong.

The planner estimated that scanning the table was cheaper than using an index.

## Lesson

Support Engineers investigate **why** a Sequential Scan was chosen rather than assuming it is a problem.

Possible reasons include:

- The table is relatively small.
- Many rows satisfy the filter.
- No useful index exists.
- The planner estimates a Sequential Scan is cheaper.

---

# Step 3 — Is PostgreSQL Using Parallel Execution?

The execution plan shows:

```
Workers Planned: 2

Workers Launched: 2
```

## Observation

PostgreSQL decided to distribute the work across multiple worker processes.

## Lesson

Parallel execution is generally a sign that PostgreSQL believes the workload is large enough to benefit from multiple CPU cores.

---

# Step 4 — Were the Planner's Estimates Reasonable?

Planner estimate:

```
rows = 60861
```

Actual execution:

```
rows = 50598
```

## Observation

The planner's estimate is reasonably close to reality.

## Lesson

Large differences between estimated and actual rows may indicate outdated statistics.

In this case, the planner appears to have made a good estimate.

---

# Step 5 — Why Was a Parallel Sequential Scan Chosen?

The execution plan shows:

```
Parallel Seq Scan

Filter:

shift_start_time IS NOT NULL
```

Actual rows scanned:

```
50598
```

## Observation

A large number of rows satisfy the filter.

Using an index may not provide a significant advantage.

## Lesson

Support Engineers should consider the selectivity of a query before recommending new indexes.

---

# Step 6 — Is Sorting Expensive?

The execution plan shows:

```
Sort Method:

quicksort

Memory:

58 kB
```

## Observation

The sort operation uses very little memory.

## Lesson

Sorting is unlikely to be the performance bottleneck for this query.

---

# Step 7 — Why Is PostgreSQL Performing an Index Scan Backward?

The execution plan shows:

```
Index Scan Backward
```

The SQL query contains:

```sql
ORDER BY js.to_date_time_timestamp DESC
```

## Observation

Instead of sorting the results after reading them, PostgreSQL walks the index in reverse order.

## Lesson

Reading an index backwards is often much more efficient than performing a separate sort operation.

---

# Step 8 — Rows Removed by Filter

The execution plan reports:

```
Rows Removed by Filter: 1
```

## Observation

PostgreSQL located matching rows using the index but filtered out rows that did not satisfy the remaining conditions.

## Lesson

Indexes help locate candidate rows.

Filters determine whether those rows satisfy the complete query.

---

# Step 9 — "Never Executed"

The execution plan reports:

```
Index Scan using nursa_user_pkey

(never executed)
```

## Observation

Earlier joins produced zero matching rows.

Because no rows remained, PostgreSQL skipped the final index lookup.

## Lesson

Execution plans stop processing unnecessary operations whenever possible.

---

# Step 10 — Planning Time vs Execution Time

Planning:

```
11.9 ms
```

Execution:

```
49.1 ms
```

## Observation

Planning consumed relatively little time.

Most of the work occurred during query execution.

## Lesson

When troubleshooting slow queries, determine whether the time is spent planning or executing.

In most production workloads, execution dominates.

---

# Overall Investigation Summary

## Good Signs

- Multiple indexes are being used.
- Parallel execution is enabled.
- Planner estimates closely match reality.
- Sorting consumes very little memory.
- Planning time is low.

---

## Interesting Observations

- PostgreSQL performs one Parallel Sequential Scan.
- Approximately 50,000 rows satisfy the filter.
- The planner appears to have chosen a reasonable execution plan.

---

## Questions Worth Investigating

Rather than immediately recommending another index, a Support Engineer should ask:

- Is the Sequential Scan actually the cheapest option?
- How selective is the filter?
- Would a composite index reduce the number of scanned rows?
- Has the workload changed recently?
- Would EXPLAIN ANALYZE on similar queries show the same pattern?

---

# Key Lessons

- EXPLAIN ANALYZE tells the story of how PostgreSQL executed a query.
- Sequential Scans are not inherently bad.
- Index usage should always be interpreted in context.
- Accurate planner estimates are a sign of healthy database statistics.
- Execution plans should be treated as evidence rather than assumptions.

---

# Support Engineer Mindset

When investigating PostgreSQL performance, avoid jumping directly to solutions.

Instead, gather evidence and ask questions.

Think like a detective.

Observe first.

Form hypotheses.

Test those hypotheses.

Only then recommend changes.

That approach leads to better troubleshooting, better communication with engineering teams, and better outcomes for customers.