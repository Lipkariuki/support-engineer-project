# Lab 01 — Tables, Pages & Tuples

> Understanding how PostgreSQL stores data internally.

---

## Objective

Before learning indexes, we first need to understand how PostgreSQL stores data on disk.

Although we write SQL using tables and rows, PostgreSQL does not physically store data that way.

Instead, PostgreSQL stores information inside **pages**, and each page contains **tuples**.

Understanding this concept explains why queries behave the way they do and why indexes, VACUUM, and query optimization are so important.

---

# Key Concepts

## Table

A table is the logical object we interact with using SQL.

Example:

```sql
SELECT * FROM users;
```

To us, it looks like one large spreadsheet.

Internally, PostgreSQL breaks this table into many smaller pieces.

---

## Page

A page is the smallest unit PostgreSQL reads and writes.

By default, every page is **8 KB**.

Think of a page like one page in a book.

A large table may contain thousands—or even millions—of pages.

---

## Tuple

A tuple is PostgreSQL's physical representation of a row.

For example:

| id | name | email |
|----|------|--------|
|1|Philip|philip@email.com|

This row is stored internally as a tuple inside one page.

---

# How Storage Works

Imagine a table with many users.

Instead of storing everything together:

Table

↓

Page 1
- Tuple
- Tuple
- Tuple

↓

Page 2
- Tuple
- Tuple
- Tuple

↓

Page 3
- Tuple
- Tuple

PostgreSQL searches through pages rather than looking directly for rows.

---

# Why Pages Matter

When PostgreSQL executes a query, it does not fetch one row from disk.

Instead, it loads an entire page into memory.

If the row you need is inside that page, PostgreSQL can access it immediately.

This design is much faster because reading from disk is expensive.

---

# Why This Matters for Support Engineers

Imagine a customer says:

> "Our database doubled in size overnight."

A Support Engineer should think beyond the number of rows.

Questions to investigate include:

- Did the number of pages increase?
- Are UPDATE operations creating dead tuples?
- Has VACUUM failed to reclaim storage?
- Is PostgreSQL reading significantly more pages than before?

Understanding pages helps explain many storage and performance issues.

---

# Questions

## 1. What is the difference between a table, a page, and a tuple?

A table is the logical structure users interact with using SQL.

A page is an 8 KB block where PostgreSQL physically stores data.

A tuple is the physical version of a row stored inside a page.

---

## 2. Why does PostgreSQL read pages instead of individual rows?

Reading an entire page is much faster than reading one row at a time.

Disk operations are expensive, so PostgreSQL minimizes them by loading complete pages into memory, allowing multiple rows to be processed with a single read.

---

## 3. If someone inserts one new row, where is it physically stored?

The new row is stored as a new tuple.

PostgreSQL places that tuple into a page that has available space.

If no page has enough free space, PostgreSQL allocates a new page.

---

## 4. Why is the 8 KB page size important?

The 8 KB page is PostgreSQL's basic storage unit.

Every table scan, index lookup, UPDATE, DELETE, and VACUUM operation works with pages.

Understanding page size helps explain query performance, storage growth, and database maintenance.

---

# Diagram

```

Database

│

├── users table

│

├── Page 1 (8 KB)

│ ├── Tuple (Philip)

│ ├── Tuple (Jane)

│ └── Tuple (John)

│

├── Page 2 (8 KB)

│ ├── Tuple (Mary)

│ ├── Tuple (James)

│ └── Tuple (Peter)

│

└── Page 3 (8 KB)

├── Tuple (Alice)

└── Tuple (Bob)

```

---

# Key Takeaways

- A table is a logical structure used in SQL.
- PostgreSQL physically stores table data in pages.
- Pages are typically **8 KB** in size.
- Rows are stored internally as tuples.
- PostgreSQL reads and writes pages, not individual rows.
- Many database performance issues are actually page-related.
- Understanding pages is the foundation for learning indexes, VACUUM, and query optimization.

---

# Support Engineer Mindset

When troubleshooting PostgreSQL, don't think only about rows.

Ask yourself:

- How many pages are being read?
- Are dead tuples increasing storage?
- Is VACUUM reclaiming space?
- Could an index reduce the number of pages PostgreSQL needs to read?

Thinking in terms of pages instead of rows is one of the biggest shifts from writing SQL to becoming an effective PostgreSQL Support Engineer.
