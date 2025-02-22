# Lesson 2: Advanced and Beginner PostgreSQL Features

### Table of Contents
1. [Beginner Level Joins](#beginner-level-joins)
   - [Inner Join](#inner-join)
   - [Left Outer Join](#left-outer-join)
   - [Right Outer Join](#right-outer-join)

2. [Window Functions](#window-functions)
   - [Syntax](#syntax)
   - [Examples](#examples)

3. [Indexes](#indexes)
   - [Types of Indexes](#types-of-indexes)
   - [Creating Indexes](#creating-indexes)

4. [Joins Beyond Basics](#joins-beyond-basics)
   - [Self Joins](#self-joins)
   - [Full Outer Joins](#full-outer-joins)
   - [Cross Joins](#cross-joins)

5. [Advanced Data Types](#advanced-data-types)
   - [JSON/JSONB](#json-jsonb)
   - [Array](#array)

6. [Performance Tuning](#performance-tuning)
   - [EXPLAIN and EXPLAIN ANALYZE](#explain-and-explain-analyze)
   - [Vacuum and Analyze](#vacuum-and-analyze)

---

## Beginner Level Joins

### Inner Join
An Inner Join returns only the rows where there is a match in both tables.
```sql
SELECT employees.name, departments.name
FROM employees
INNER JOIN departments ON employees.department_id = departments.id;
```

### Left Outer Join
A Left Outer Join returns all rows from the left table and matching rows from the right table. Rows without a match in the right table return NULL.
```sql
SELECT employees.name, departments.name
FROM employees
LEFT JOIN departments ON employees.department_id = departments.id;
```

### Right Outer Join
A Right Outer Join returns all rows from the right table and matching rows from the left table. Rows without a match in the left table return NULL.
```sql
SELECT employees.name, departments.name
FROM employees
RIGHT JOIN departments ON employees.department_id = departments.id;
```

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

