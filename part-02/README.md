# 📌 PART 2: DDL (Data Definition Language) & Constraints

## 1. Primary Key Auto-generation Strategies

In PostgreSQL, there are multiple ways to auto-generate primary keys. Choosing the right one depends on your use case.

### 1️⃣ SERIAL (Old Way)

```sql
CREATE TABLE course (
    course_id SERIAL PRIMARY KEY
);
```

- Internally creates a sequence and sets DEFAULT nextval(...).

- Pros: Simple, widely used.

- Cons: Not SQL standard, can cause issues in distributed systems.

### 2️⃣ BIGSERIAL (For Large Tables)

```sql
CREATE TABLE course (
    course_id BIGSERIAL PRIMARY KEY
);
```

- Same as SERIAL, but uses a larger integer range (8 bytes).

- Use when expecting more than 2 billion rows.

### 3️⃣ Identity (Manual Sequence + Default)

```sql
CREATE TABLE course (
    course_id INT PRIMARY KEY DEFAULT nextval('course_id_seq')
);
```

- Can control sequence manually (reset, increment, change).

- Flexible but requires extra setup.

### 4️⃣ IDENTITY (Modern SQL Standard - PostgreSQL 10+)

```sql
CREATE TABLE course (
    course_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
);
```

- Cleaner syntax, sequence is internally managed.

- Best Practice: Use this for modern applications.

### 5️⃣ UUID (Universally Unique Identifier)

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE course (
    course_id UUID PRIMARY KEY DEFAULT uuid_generate_v4()
);
```

- Best for distributed systems where sequential numbers aren't unique across nodes.

- Collision chance is extremely low.

- Cons: Slightly larger storage size (16 bytes).

### 📝 Summary Table

 |#   | Use Case              | Recommended Type                       |
| --- | ------------------ | --------------------------- |
| 01  | Small Table             | `SERIAL` |  
| 01  | Big Table             | `BIGSERIAL` |  
| 01  | Modern Standard            | `IDENTITY` |  
| 01  | Custom Control           | `Manual Sequence` |  
| 01  | Distributed System             | `UUID` |  


---

## 2. Create Table

###  1️⃣ Column Level Constraint
```sql
CREATE TABLE table_name (
    col_name data_type CONSTRAINT_TYPE
);
```

```sql
-- Example:
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE
);
```

### 2️⃣ Table Level Constraint

```sql
CREATE TABLE table_name (
    col1 data_type,
    col2 data_type,
    CONSTRAINT constraint_name CONSTRAINT_TYPE (col1, col2)
);
```

```sql
-- Example:
CREATE TABLE departments (
    dept_id INT,
    dept_name VARCHAR(50),
    CONSTRAINT pk_dept PRIMARY KEY (dept_id)
);
```

---

## 3. Alter Table

### Add Column

```sql
ALTER TABLE table_name ADD COLUMN col_name data_type;
```

### Drop Column

```sql
ALTER TABLE table_name DROP COLUMN col_name;
```

### Modify Column Data Type

```sql
ALTER TABLE table_name ALTER COLUMN col_name TYPE data_type;
```

>Note: PostgreSQL uses `ALTER COLUMN ... TYPE`, not `MODIFY`.

### Add Constraints

```sql
ALTER TABLE employees ADD CONSTRAINT unique_email UNIQUE (email);

ALTER TABLE employees ADD CONSTRAINT chk_salary CHECK (salary > 0);

ALTER TABLE employees ADD CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departments(dept_id);
```

### Drop Constraints

```sql
ALTER TABLE employees DROP CONSTRAINT fk_dept;
```

### Replace a Constraint (Step-by-Step)
1. Drop the old constraint.
2. Add the new constraint.

```sql
ALTER TABLE employees DROP CONSTRAINT fk_dept;

ALTER TABLE employees ADD CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departments(dept_id);
```
 
---

## 4. Drop / Truncate / Rename

### Drop Table

```sql
DROP TABLE table_name;
```
- Irreversible.
- Deletes both structure and data.

### Truncate Table

```sql
TRUNCATE TABLE table_name;
```

- Deletes all data but keeps the structure.
- Faster than DELETE.
- Structure Safe.

### Rename Table

```sql
ALTER TABLE student RENAME TO learners;
```

### 🆚 TRUNCATE vs DELETE vs DROP

| Feature   | TRUNCATE  | DELETE | DROP |
| --------- | --------- | ------- | ------ |
| Removes Data  | Yes   | Yes |Yes |
| Removes Structure  | No   | No |Yes |
| WHERE Clause | No   | Yes | No |
| Rollback Possible  | Yes (in transaction)   | Yes |No |
| Speed | Fast   | Slow | Fast |
| Triggers Fired  | No   | Yes | No |

---

## 5. Indexes

### Create Index

```sql
CREATE INDEX idx_name ON table_name(col1);
```

### Drop Index

```sql
DROP INDEX idx_name;
```

### Types of Indexes (Bonus)
- B-Tree: Default, good for equality and range queries.
- Hash: Good for equality comparisons only.
- GIN: Good for JSONB, arrays, and full-text search.
- GiST: Good for geometric data and full-text search.
- BRIN: Good for large, naturally ordered data.
- Partial Index: Index only a subset of rows.

---

## 6. Constraints Deep Dive

### 📊 Constraint Level Matrix

| Constraint   | Column Level  | Table Level | ALTER TABLE |  Note |
| --------- | --------- | ------- | ------ | ---- |
| NOT NULL  | ✅   | [x] |✅ | Column level best. |
| UNIQUE  | ✅   | ✅ |✅ | Table level for composite keys. |
| PRIMARY KEY	  | ✅   | ✅ |✅ | Table level for composite keys. |
| DEFAULT  | ✅   | [x] |✅ | Only single column. |
| CHECK | ✅   | ✅ |✅ | Column (single), Table (multi-col). |
| FOREIGN KEY | ✅   | ✅ |✅ | Table level preferred for clarity. |


```sql
-- Example: Column Level Constraint
CREATE TABLE table_name (
    col_name INT PRIMARY KEY,
    col_name INT REFERENCES table_name(id)
);
```


```sql
-- Example: Table Level Constraint
CREATE TABLE employees (
    emp_id INT,
    dept_id INT,
    salary DECIMAL(10, 2),
    CONSTRAINT pk_emp PRIMARY KEY (emp_id),
    CONSTRAINT fk_emp FOREIGN KEY (dept_id) REFERENCES departments(dept_id),
    CONSTRAINT chk_salary CHECK (salary > 1000)
);
```

```sql
-- Altering Constraints

-- Modify column
ALTER TABLE employees ALTER COLUMN email SET NOT NULL;

-- Add constraints
ALTER TABLE employees ADD CONSTRAINT unique_email UNIQUE (email);
ALTER TABLE employees ADD CONSTRAINT pk_emp PRIMARY KEY (id);
ALTER TABLE employees ADD CONSTRAINT chk_salary CHECK (salary > 0);
```

### Pros & Cons of Altering Constraints
- Pro: Existing table can also be altered.
- Cons: If data already violates the constraint, it will show an error.

---

## 7. Bonus: CASCADE vs RESTRICT

When you drop a parent table, what happens to the child tables?
- CASCADE: Automatically drops dependent objects.

```sql
DROP TABLE departments CASCADE;
```

- RESTRICT: Prevents dropping if dependent objects exist.

```sql
DROP TABLE departments RESTRICT;
```

- SET NULL: Sets the foreign key column to `NULL` when the parent is deleted.

```sql
ALTER TABLE employees
DROP CONSTRAINT fk_dept,
ADD CONSTRAINT fk_dept
FOREIGN KEY (dept_id) REFERENCES departments(dept_id) ON DELETE SET NULL;
```

---

## 8. Bonus: Temporary Tables vs Unlogged Tables

### Temporary Tables
```sql
CREATE TEMP TABLE temp_data (
    id INT,
    name VARCHAR(50)
);
```
- Exists only for the duration of the session.
- Automatically dropped when the session ends.

### Unlogged Tables

```sql
CREATE UNLOGGED TABLE fast_data (
    id INT,
    name VARCHAR(50)
);
```
- Faster than regular tables (no WAL writing).
- Not crash-safe: Data is truncated after a crash.
- Good for temporary data processing.

---
[< Previous](../part-01/README.md) ---- [Next >](../part-03/README.md)