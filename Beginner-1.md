# PostgreSQL Concepts by Skill Level

### Table of Contents
- [Beginner Level](#beginner-level)
  - Data Definition Language (DDL)
  - Data Manipulation Language (DML)
  - Constraints
- [Intermediate Level](#intermediate-level)
  - Views and Materialized Views
  - User Roles and Permissions
  - Stored Procedures and Functions
- [Advanced Level](#advanced-level)
  - Triggers
  - Advanced Query Optimization
  - Concurrency Control
  - Backup and Restore

---

## Beginner Level
### Data Definition Language (DDL)
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    department_id INT,
    hire_date DATE
);
ALTER TABLE employees ADD COLUMN salary NUMERIC;
DROP TABLE employees;
```

### Data Manipulation Language (DML)
```sql
INSERT INTO employees (name, department_id, hire_date)
VALUES ('John Doe', 1, '2023-01-01');
UPDATE employees SET salary = 60000 WHERE id = 1;
DELETE FROM employees WHERE id = 1;
```

### Constraints
```sql
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
ALTER TABLE employees
ADD CONSTRAINT fk_department FOREIGN KEY (department_id)
REFERENCES departments(id);
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE
);
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price NUMERIC CHECK (price > 0)
);
```

---

## Intermediate Level
### Views and Materialized Views
```sql
CREATE VIEW employee_view AS
SELECT name, department_id FROM employees;
CREATE MATERIALIZED VIEW employee_summary AS
SELECT department_id, COUNT(*) FROM employees GROUP BY department_id;
```

### User Roles and Permissions
```sql
CREATE ROLE admin LOGIN PASSWORD 'password';
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin;
```

### Stored Procedures and Functions
```sql
CREATE FUNCTION get_employee_count() RETURNS INT AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM employees);
END;
$$ LANGUAGE plpgsql;
```

---

## Advanced Level
### Triggers
```sql
CREATE TRIGGER update_salary
AFTER UPDATE OF salary ON employees
FOR EACH ROW EXECUTE FUNCTION log_salary_change();
```

### Advanced Query Optimization
```sql
CLUSTER employees USING idx_name;
REINDEX TABLE employees;
```

### Concurrency Control
```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
LOCK TABLE employees IN EXCLUSIVE MODE;
```

### Backup and Restore
```bash
pg_dump mydatabase > backup.sql
psql mydatabase < backup.sql
```

