# Lab 02 — MVCC & Dead Tuples

> Understanding how PostgreSQL manages concurrent updates without overwriting existing rows.

---

## Objective

One of PostgreSQL's greatest strengths is its ability to allow many users to read and write data at the same time.

Instead of modifying an existing row during an `UPDATE` or `DELETE`, PostgreSQL creates a **new version** of that row.

This behavior is called **Multi-Version Concurrency Control (MVCC)**.

Understanding MVCC explains why PostgreSQL needs **VACUUM**, why **dead tuples** exist, and why PostgreSQL performs so well under concurrent workloads.

---

# Key Concepts

## MVCC

MVCC stands for **Multi-Version Concurrency Control**.

Instead of overwriting an existing row, PostgreSQL creates a new version whenever a row is updated.

This allows multiple transactions to safely read and write data without constantly blocking each other.

---

## Tuple

A tuple is PostgreSQL's physical representation of a row.

For example:

| id | name |
|----|------|
| 1 | Philip |

Internally, this row is stored as a tuple inside a database page.

Whenever the row is updated, PostgreSQL creates another tuple instead of replacing the existing one.

---

## UPDATE

Consider the following query:

```sql
UPDATE users
SET name = 'Phil'
WHERE id = 1;
```

Many people assume PostgreSQL simply changes the value from **Philip** to **Phil**.

Instead, PostgreSQL:

1. Creates a new tuple containing the updated data.
2. Marks the previous tuple as obsolete.
3. Makes the new tuple visible to future transactions.

The original tuple remains on disk until PostgreSQL can safely remove it.

---

## Dead Tuple

A dead tuple is an old version of a row that is no longer needed by any active transaction.

Dead tuples continue occupying space inside database pages until PostgreSQL removes them.

This cleanup process is performed by **VACUUM**, which we will cover in the next lab.

---

# How MVCC Works

Imagine the following row exists.

| id | name |
|----|------|
| 1 | Philip |

An update is executed.

```sql
UPDATE users
SET name = 'Phil'
WHERE id = 1;
```

Internally PostgreSQL creates a new tuple instead of modifying the original one.

```
Original Tuple

id = 1
name = Philip

        │
        │ UPDATE
        ▼

New Tuple

id = 1
name = Phil
```

The application only sees the newest visible tuple.

The previous tuple eventually becomes a dead tuple once it is no longer required by any transaction.

---

# Why PostgreSQL Uses MVCC

Imagine two users accessing the same row.

- Alice is reading customer information.
- Bob updates that customer's name.

Without MVCC:

- Alice might be blocked while Bob performs the update.
- Bob might have to wait until Alice finishes reading.

With MVCC:

- Alice continues reading the original tuple.
- Bob creates a new tuple.
- Both transactions continue without interfering with each other.

This approach greatly improves concurrency while reducing unnecessary locking.

---

# Why Dead Tuples Exist

Every time PostgreSQL updates a row, it creates a new tuple.

The previous version is no longer needed by future transactions.

However, PostgreSQL cannot immediately remove it because another active transaction might still be reading that older version.

Once PostgreSQL determines that no active transaction needs the old tuple anymore, it becomes a **dead tuple** and is ready to be cleaned up by VACUUM.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "Our database keeps getting larger even though we rarely insert new rows."

A Support Engineer should immediately consider questions such as:

- Are rows being updated frequently?
- Are dead tuples accumulating?
- Is autovacuum running correctly?
- Is table bloat increasing?
- Are long-running transactions preventing cleanup?

Understanding MVCC is the first step toward investigating these issues.

---

# Knowledge Check

## 1. What is MVCC?

MVCC (Multi-Version Concurrency Control) is PostgreSQL's mechanism for handling concurrent transactions by creating new row versions instead of overwriting existing ones.

---

## 2. Why doesn't PostgreSQL overwrite rows during an UPDATE?

Overwriting a row could interfere with transactions that are still reading the previous version.

Creating a new tuple allows readers and writers to operate simultaneously without blocking each other.

---

## 3. What is a dead tuple?

A dead tuple is an old version of a row that is no longer visible to any active transaction.

It remains on disk until VACUUM removes it.

---

## 4. Why doesn't PostgreSQL remove dead tuples immediately?

Another active transaction may still need to read the previous version of the row.

PostgreSQL waits until it is safe before reclaiming the storage space.

---

## 5. How does MVCC improve performance?

MVCC reduces blocking between transactions by allowing readers and writers to work on different versions of the same row.

This improves concurrency and overall database performance.

---

# Diagram

```text
Database

│

└── users table

    │

    └── Page

        │

        ├── Tuple (Original)

        │     id = 1

        │     name = Philip

        │

        ├── Tuple (Updated)

        │     id = 1

        │     name = Phil

        │

        └── Dead Tuple

                │

                ▼

             VACUUM
```

---

# Key Takeaways

- MVCC stands for **Multi-Version Concurrency Control**.
- PostgreSQL does **not** overwrite rows during an UPDATE.
- Every UPDATE creates a **new tuple**.
- Older row versions eventually become **dead tuples**.
- Dead tuples continue occupying storage until VACUUM removes them.
- MVCC allows many users to access the same database simultaneously with minimal locking.
- Understanding MVCC is essential before learning VACUUM, table bloat, and query optimization.

---

# Support Engineer Mindset

When troubleshooting PostgreSQL, don't stop at the UPDATE statement.

Instead, ask yourself:

- Is this table updated frequently?
- Could dead tuples be increasing storage usage?
- Is autovacuum keeping up with cleanup?
- Could table bloat be affecting performance?
- Are long-running transactions preventing dead tuples from being removed?

Thinking in terms of **row versions** rather than simply **rows** is one of the biggest mindset shifts from writing SQL to becoming an effective PostgreSQL Support Engineer.

---

# Looking Ahead

In the next lab, we'll answer an important question:

> **If PostgreSQL never immediately deletes old tuples, what eventually cleans them up?**

That leads us directly into **VACUUM & Autovacuum**, one of the most important topics for every PostgreSQL Support Engineer.