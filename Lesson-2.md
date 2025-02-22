# Lesson 2: Advanced PostgreSQL Features

### Table of Contents
1. [Window Functions](#window-functions)
   - [Syntax](#syntax)
   - [Examples](#examples)

2. [Indexes](#indexes)
   - [Types of Indexes](#types-of-indexes)
   - [Creating Indexes](#creating-indexes)

3. [Joins Beyond Basics](#joins-beyond-basics)
   - [Self Joins](#self-joins)
   - [Full Outer Joins](#full-outer-joins)
   - [Cross Joins](#cross-joins)

4. [Advanced Data Types](#advanced-data-types)
   - [JSON/JSONB](#json-jsonb)
   - [Array](#array)

5. [Performance Tuning](#performance-tuning)
   - [EXPLAIN and EXPLAIN ANALYZE](#explain-and-explain-analyze)
   - [Vacuum and Analyze](#vacuum-and-analyze)

---

## Window Functions
Window functions perform calculations across a set of table rows related to the current row.

### Syntax
```sql
SELECT column_name,
       window_function() OVER (PARTITION BY column_name ORDER BY column_name) AS alias
FROM table_name;
```

### Examples
```sql
SELECT employee_id, department_id, salary,
       RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS salary_rank
FROM employees;
```

---

## Indexes
Indexes improve the speed of data retrieval operations.

### Types of Indexes
- **B-tree** (default, suitable for equality and range queries)
- **Hash** (fast equality comparisons)
- **GIN** (for full-text search and array values)
- **GiST** (useful for geometric data)

### Creating Indexes
```sql
CREATE INDEX idx_name ON table_name (column_name);
CREATE UNIQUE INDEX idx_unique_name ON table_name (column_name);
CREATE INDEX idx_gin_name ON table_name USING GIN (column_name);
```

---

## Joins Beyond Basics
### Self Joins
```sql
SELECT A.employee_name, B.employee_name AS manager_name
FROM employees A
JOIN employees B ON A.manager_id = B.employee_id;
```

### Full Outer Joins
```sql
SELECT *
FROM table_a
FULL OUTER JOIN table_b ON table_a.id = table_b.id;
```

### Cross Joins
```sql
SELECT *
FROM table_a
CROSS JOIN table_b;
```

---

## Advanced Data Types
### JSON/JSONB
```sql
SELECT data->>'name' AS name
FROM json_table;
```

### Array
```sql
SELECT unnest(array_column)
FROM array_table;
```

---

## Performance Tuning
### EXPLAIN and EXPLAIN ANALYZE
```sql
EXPLAIN SELECT * FROM large_table;
EXPLAIN ANALYZE SELECT * FROM large_table;
```

### Vacuum and Analyze
```sql
VACUUM ANALYZE;
VACUUM FULL;
```

