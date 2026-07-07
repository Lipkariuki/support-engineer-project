# Lab 03 — VACUUM & Autovacuum

> Understanding how PostgreSQL cleans up dead tuples and keeps tables healthy.

---

## Objective

In the previous lab, we learned that PostgreSQL does not overwrite rows during an UPDATE.

Instead, PostgreSQL creates a new tuple while the old tuple eventually becomes a **dead tuple**.

This raises an important question:

> **If PostgreSQL never immediately deletes old tuples, what eventually cleans them up?**

The answer is **VACUUM**.

Understanding VACUUM is essential for maintaining PostgreSQL performance, preventing table bloat, and troubleshooting storage-related issues.

---

# Key Concepts

## VACUUM

VACUUM is a PostgreSQL maintenance operation that scans tables for dead tuples.

Instead of deleting table files, VACUUM marks the space occupied by dead tuples as **available for reuse**.

This allows PostgreSQL to insert new tuples into existing pages instead of continuously allocating new storage.

---

## Autovacuum

Autovacuum is PostgreSQL's automatic background process that runs VACUUM when necessary.

It continuously monitors tables and starts cleanup once enough dead tuples have accumulated.

Most PostgreSQL databases rely on autovacuum rather than manual VACUUM operations.

---

## Table Bloat

Table bloat occurs when a table contains many dead tuples that have not yet been cleaned up.

Although these tuples are no longer visible to users, they still consume disk space.

Excessive table bloat can lead to:

- Increased storage usage
- Slower sequential scans
- Less efficient indexes
- Reduced overall database performance

---

# How VACUUM Works

Imagine the following sequence.

```
Original Tuple

↓

UPDATE

↓

New Tuple Created

↓

Old Tuple becomes Dead

↓

VACUUM scans the page

↓

Dead tuple removed

↓

Space becomes available for reuse
```

Notice something important.

VACUUM does **not** normally shrink the size of the table.

Instead, it makes the freed space available for future INSERT and UPDATE operations.

---

# Why PostgreSQL Doesn't Delete Dead Tuples Immediately

You may wonder:

> Why not simply delete the old tuple immediately?

The answer is concurrency.

Another active transaction may still be reading the previous version of the row.

Deleting it too early could cause inconsistent query results.

Instead, PostgreSQL waits until it is safe before allowing VACUUM to reclaim the space.

---

# VACUUM vs DELETE

Many beginners assume that running a DELETE immediately frees disk space.

That is not how PostgreSQL works.

```
DELETE

↓

Dead Tuple

↓

VACUUM

↓

Reusable Space
```

DELETE only marks a row as obsolete.

VACUUM performs the cleanup later.

---

# Why This Matters for Support Engineers

Imagine a customer reports:

> "Our PostgreSQL database keeps growing every week."

Possible causes include:

- Heavy UPDATE activity
- Heavy DELETE activity
- Dead tuples accumulating
- Autovacuum not keeping up
- Long-running transactions preventing cleanup

Understanding VACUUM allows a Support Engineer to investigate these possibilities instead of assuming the database is "leaking storage."

---

# Knowledge Check

## 1. What is VACUUM?

VACUUM scans database pages, identifies dead tuples, and marks their storage space as available for reuse.

---

## 2. What is Autovacuum?

Autovacuum is PostgreSQL's automatic background worker that runs VACUUM whenever tables require maintenance.

---

## 3. Does VACUUM reduce the size of a table?

Usually, no.

VACUUM reclaims space for reuse but normally does not reduce the physical size of the table on disk.

---

## 4. What causes table bloat?

Table bloat occurs when dead tuples accumulate faster than they are cleaned up.

---

## 5. Why can long-running transactions prevent VACUUM?

Because PostgreSQL cannot remove row versions that may still be visible to active transactions.

---

# Diagram

```text
Page

│

├── Live Tuple

├── Dead Tuple

├── Live Tuple

├── Dead Tuple

│

▼

VACUUM

│

▼

Reusable Free Space

│

▼

Future INSERT reuses the space
```

---

# Key Takeaways

- VACUUM cleans up dead tuples created by UPDATE and DELETE operations.
- VACUUM normally does not reduce the physical size of a table.
- Freed space becomes available for future inserts and updates.
- Autovacuum automatically performs routine maintenance in the background.
- Table bloat occurs when dead tuples accumulate.
- Long-running transactions can delay cleanup.
- Healthy autovacuum behavior is critical for PostgreSQL performance.

---

# Support Engineer Mindset

When investigating PostgreSQL performance issues, don't immediately assume the database is broken.

Instead, ask yourself:

- Is autovacuum running?
- Are dead tuples accumulating?
- Is table bloat increasing?
- Are long-running transactions blocking cleanup?
- Has the workload recently shifted toward more UPDATE or DELETE operations?

Many PostgreSQL storage and performance problems are actually maintenance problems rather than hardware problems.

---

# Looking Ahead

In the next lab, we'll explore **Indexes**.

We'll answer questions such as:

- What is an index?
- How does an index help PostgreSQL find rows faster?
- Why do indexes improve read performance but slow down writes?
- Why does PostgreSQL sometimes ignore an index altogether?

Understanding indexes will complete another major piece of PostgreSQL's performance model.