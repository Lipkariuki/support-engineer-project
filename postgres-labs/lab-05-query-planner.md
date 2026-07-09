# Lab 05 — Query Planner

> Understanding how PostgreSQL decides the best way to execute a query.

---

## Objective

Every SQL query sent to PostgreSQL can be executed in multiple ways.

For example, PostgreSQL may choose to:

- Perform a Sequential Scan
- Use an Index Scan
- Join tables in different orders

How does PostgreSQL decide which option is best?

The answer is the **Query Planner**.

By the end of this lab, you'll understand how PostgreSQL estimates query cost, why it sometimes ignores indexes, and why understanding the planner is essential for troubleshooting database performance.

---

# Key Concepts

## Query Planner

The Query Planner is PostgreSQL's decision-making engine.

Every SQL query passes through the planner before execution.

Its job is to evaluate different execution strategies and choose the one with the lowest estimated cost.

The planner does **not** always choose an Index Scan.

Instead, it chooses the plan that it estimates will execute most efficiently.

---

## Cost

When viewing an execution plan, you will often see something like:

```text
cost=0.43..120.56
```

This is **not** the execution time.

Cost is PostgreSQL's internal estimate of how expensive a query plan is.

The planner compares the estimated cost of multiple execution plans and selects the lowest one.

---

## Statistics

The Query Planner makes decisions using database statistics.

These statistics include information such as:

- Estimated number of rows
- Number of distinct values
- Distribution of data
- Percentage of NULL values

Rather than counting every row before executing a query, PostgreSQL uses these statistics to estimate the cheapest execution plan.

---

# How the Query Planner Works

Imagine the following query.

```sql
SELECT *
FROM users
WHERE email = 'philip@example.com';
```

The planner considers different ways to execute it.

```
Option 1

Sequential Scan

Estimated Cost = 150

----------------------------

Option 2

Index Scan

Estimated Cost = 12
```

Since the Index Scan has the lower estimated cost, PostgreSQL chooses it.

Now consider another query.

```sql
SELECT *
FROM users;
```

Possible execution plans:

```
Sequential Scan

Estimated Cost = 120

----------------------------

Index Scan

Estimated Cost = 850
```

Because every row must be returned, PostgreSQL chooses a Sequential Scan instead.

---

# Why PostgreSQL Doesn't Always Use an Index

Many people assume that once an index exists, PostgreSQL should always use it.

This is incorrect.

The planner evaluates whether using the index is actually beneficial.

Some common reasons PostgreSQL may ignore an index include:

## 1. The Table Is Small

If a table contains only a few rows, reading the entire table is often faster than using an index.

---

## 2. The Query Returns Most Rows

Suppose the following query returns 95% of the table.

```sql
SELECT *
FROM users
WHERE country = 'Kenya';
```

Using an index would still require PostgreSQL to retrieve almost every row.

A Sequential Scan is usually cheaper.

---

## 3. Statistics Are Outdated

If PostgreSQL's statistics no longer reflect the current data, the planner may choose an inefficient execution plan.

Running ANALYZE updates these statistics.

---

## 4. The Query Doesn't Match the Index

Suppose an index exists on:

```sql
email
```

But the query is:

```sql
SELECT *
FROM users
WHERE lower(email) = 'philip@example.com';
```

The planner may not be able to use the standard index because the query applies a function to the indexed column.

---

## 5. The Planner Estimates a Sequential Scan Is Cheaper

Even when an index exists, PostgreSQL will ignore it if its estimates indicate that a Sequential Scan requires fewer resources.

---

# Why Statistics Matter

The Query Planner relies heavily on statistics.

Without accurate statistics, PostgreSQL may choose a poor execution plan.

Statistics are updated using:

```sql
ANALYZE users;
```

Understanding statistics becomes especially important when troubleshooting slow queries.

---

# Why This Matters for Support Engineers

Imagine a customer says:

> "We created an index yesterday, but PostgreSQL still performs a Sequential Scan."

A Support Engineer should investigate before assuming something is wrong.

Questions to ask include:

- Is the table large enough?
- Does the query return most of the table?
- Are planner statistics current?
- Does the query match the index?
- Is PostgreSQL choosing the cheapest execution plan?

Understanding the planner allows Support Engineers to explain PostgreSQL's decisions rather than guessing.

---

# Knowledge Check

## 1. What is the Query Planner?

The Query Planner evaluates different execution strategies and selects the one with the lowest estimated cost.

---

## 2. Does PostgreSQL always use indexes?

No.

PostgreSQL only uses an index when the planner estimates that doing so will be cheaper than another execution plan.

---

## 3. What does `cost=0.43..120.56` represent?

It is PostgreSQL's internal estimate of how expensive a query plan is.

It is **not** the query's execution time.

---

## 4. Name three reasons PostgreSQL may ignore an index.

- The table is very small.
- The query returns most of the rows.
- Statistics are outdated.

---

## 5. Why are statistics important?

The Query Planner uses statistics to estimate row counts and choose the most efficient execution plan.

---

# Diagram

```text
SQL Query

        │

        ▼

      Parser

        │

        ▼

  Query Planner

        │

 ┌──────┴──────┐

 ▼             ▼

Sequential   Index
   Scan       Scan

        │

        ▼

Lowest Estimated Cost

        │

        ▼

     Executor
```

---

# Key Takeaways

- Every SQL query passes through the Query Planner.
- The planner compares multiple execution strategies.
- PostgreSQL selects the plan with the lowest estimated cost.
- PostgreSQL does not always use indexes.
- Statistics play a major role in planner decisions.
- Outdated statistics can result in inefficient execution plans.
- Understanding the planner is essential for query optimization.

---

# Support Engineer Mindset

When troubleshooting slow queries, don't immediately assume PostgreSQL is making the wrong decision.

Instead, investigate methodically.

Ask yourself:

- How large is the table?
- How many rows match the query?
- Are planner statistics current?
- Does the query match the index?
- Is PostgreSQL choosing the cheapest execution plan based on the available information?

Support Engineers investigate evidence before forming conclusions.

---

# Looking Ahead

In the next lab, we'll learn how to inspect the Query Planner's decisions using one of PostgreSQL's most powerful tools:

> **EXPLAIN & EXPLAIN ANALYZE**

You'll learn how to read execution plans, identify bottlenecks, and understand why PostgreSQL chose a particular execution strategy.

By the end of the next lab, you'll be able to confidently interpret execution plans like the ones you've already encountered while working on the Nursa production database.