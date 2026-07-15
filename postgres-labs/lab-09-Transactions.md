# Lab 09 — Transactions

> Understanding how PostgreSQL guarantees that your data remains consistent.

---

⏱ **Estimated Time:** 25–30 minutes

🎯 **Difficulty:** Intermediate

📚 **Prerequisites:**

- Lab 02 — MVCC & Dead Tuples
- Lab 03 — VACUUM & Autovacuum
- Lab 08 — Locks & Concurrency

---

## Objective

A transaction allows PostgreSQL to treat multiple SQL statements as **one unit of work**.

Either every statement succeeds...

or none of them do.

By the end of this lab, you'll understand:

- What a transaction is
- Why PostgreSQL uses transactions
- What `BEGIN`, `COMMIT`, and `ROLLBACK` do
- How transactions relate to MVCC, locks, and VACUUM

---

# Key Concepts

## Transaction

A transaction is a group of SQL statements executed as a single unit.

If every statement succeeds, the transaction is committed.

If something fails, the transaction can be rolled back.

---

## BEGIN

`BEGIN` starts a new transaction.

Example:

```sql
BEGIN;
```

Changes made after `BEGIN` are temporary until they are committed.

---

## COMMIT

`COMMIT` permanently saves all changes made during the transaction.

Example:

```sql
COMMIT;
```

Once committed:

- Changes become visible to other sessions.
- Locks are released.
- The transaction ends.

---

## ROLLBACK

`ROLLBACK` cancels all changes made during the current transaction.

Example:

```sql
ROLLBACK;
```

The database returns to the state it was in before the transaction began.

---

# Example

Without a transaction:

```sql
UPDATE accounts
SET balance = balance - 10000
WHERE id = 1;

-- Database crashes here

UPDATE accounts
SET balance = balance + 10000
WHERE id = 2;
```

Money disappears.

---

Using a transaction:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 10000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 10000
WHERE id = 2;

COMMIT;
```

If an error occurs before `COMMIT`, PostgreSQL can safely roll everything back.

---

# Transactions and MVCC

When a row is updated:

```
Old Tuple

↓

New Tuple
```

The old version remains available while other transactions are still using it.

This is why MVCC depends on transactions.

---

# Transactions and Locks

When a transaction modifies a row:

```
BEGIN

↓

UPDATE Row

↓

Lock Acquired

↓

COMMIT

↓

Lock Released
```

Locks remain active until the transaction finishes.

---

# Long-Running Transactions

Suppose someone runs:

```sql
BEGIN;

UPDATE users
SET email = 'new@example.com'
WHERE id = 10;

-- Leaves for lunch ☕
```

The transaction remains open.

Possible consequences:

- Locks stay active.
- Other transactions may wait.
- VACUUM cannot clean certain dead tuples.
- Storage usage may increase.

Long-running transactions are a common production issue.

---

# ACID Properties

Every PostgreSQL transaction follows the ACID principles.

## Atomicity

All operations succeed, or none do.

---

## Consistency

The database always remains in a valid state.

---

## Isolation

Transactions do not interfere with one another unexpectedly.

---

## Durability

Once committed, changes survive crashes and system failures.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "Our application is hanging."

Possible causes include:

- A transaction was left open.
- A lock is blocking another transaction.
- VACUUM cannot reclaim dead tuples.
- Other sessions are waiting for the transaction to finish.

Understanding transactions helps explain all of these scenarios.

---

# Knowledge Check

## 1. What is a transaction?

A transaction is a group of SQL statements treated as one unit of work.

---

## 2. What does BEGIN do?

It starts a new transaction.

---

## 3. What does COMMIT do?

It permanently saves all changes made during the transaction.

---

## 4. What does ROLLBACK do?

It cancels all uncommitted changes.

---

## 5. Why are long-running transactions dangerous?

Because they can:

- Hold locks
- Block other queries
- Prevent VACUUM from cleaning dead tuples
- Increase storage usage

---

# Diagram

```text
BEGIN

   │

   ▼

Statement 1

   │

Statement 2

   │

Statement 3

   │

──────────────

Success?

   │

Yes ─────────► COMMIT

   │

No

   ▼

ROLLBACK
```

---

# How Everything Connects

```text
Transaction

      │

      ▼

UPDATE Row

      │

      ▼

MVCC

Creates New Tuple

      │

      ▼

Old Tuple Becomes Dead

      │

      ▼

VACUUM Cleans It Later

      │

      ▼

Locks Protect The Update

      │

      ▼

COMMIT Releases Locks
```

---

# Key Takeaways

- Transactions group multiple SQL statements into one unit of work.
- `BEGIN` starts a transaction.
- `COMMIT` permanently saves changes.
- `ROLLBACK` discards uncommitted changes.
- Transactions work together with MVCC and locks.
- Long-running transactions can block queries and delay VACUUM.

---

# Support Engineer Mindset

When investigating database performance, ask yourself:

- Is there a long-running transaction?
- Is it holding locks?
- Are other sessions waiting?
- Is VACUUM unable to clean dead tuples?
- Has the application forgotten to commit?

Investigate the transaction before assuming the database is slow.

---

# 🔍 What You'll See in Production

Common support issues include:

- Applications forgetting to `COMMIT`
- Transactions left open for long periods
- Blocking caused by active transactions
- Dead tuples accumulating because old transactions remain open
- API requests waiting on locked rows

---

# Looking Ahead

Next we'll learn one of PostgreSQL's most important internal components:

> **Lab 10 — Write-Ahead Logging (WAL)**

You'll learn:

- Why PostgreSQL writes to WAL before writing to tables
- How crash recovery works
- Why WAL files grow
- Why WAL is essential for replication and backups

---

# 🎯 Interview Takeaway

After completing this lab, I can confidently explain:

- What a PostgreSQL transaction is.
- The purpose of `BEGIN`, `COMMIT`, and `ROLLBACK`.
- How transactions protect data consistency.
- How transactions relate to MVCC, locks, and VACUUM.
- Why long-running transactions are a common production issue.