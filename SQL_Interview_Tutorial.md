# SQL Interview Tutorial - Quick Recap Guide

## Table of Contents
1. [SQL Basics](#sql-basics)
2. [SELECT Statements](#select-statements)
3. [WHERE Clause & Filtering](#where-clause--filtering)
4. [JOIN Operations](#join-operations)
5. [Aggregate Functions](#aggregate-functions)
6. [GROUP BY & HAVING](#group-by--having)
7. [Subqueries](#subqueries)
8. [Window Functions](#window-functions)
9. [Indexes](#indexes)
10. [Transactions & ACID](#transactions--acid)
11. [Normalization](#normalization)
12. [Common Interview Questions](#common-interview-questions)

---
## SQL Basics

### What is SQL?
SQL (Structured Query Language) is a standard language for managing and manipulating relational databases.

### Key SQL Commands Categories
- **DDL (Data Definition Language)**: CREATE, ALTER, DROP, TRUNCATE
- **DML (Data Manipulation Language)**: SELECT, INSERT, UPDATE, DELETE
- **DCL (Data Control Language)**: GRANT, REVOKE
- **TCL (Transaction Control Language)**: COMMIT, ROLLBACK, SAVEPOINT

### Example: Creating a Table
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,
    department VARCHAR(50),
    salary DECIMAL(10, 2),
    hire_date DATE
);
```

### Interview Question 1: What's the difference between DELETE, TRUNCATE, and DROP?

**Answer:**
- **DELETE**: Removes rows one by one, can use WHERE clause, can be rolled back, slower, triggers are fired
  ```sql
  DELETE FROM employees WHERE department = 'Sales';
  ```

- **TRUNCATE**: Removes all rows at once, cannot use WHERE clause, cannot be rolled back (in most databases), faster, triggers are not fired
  ```sql
  TRUNCATE TABLE employees;
  ```

- **DROP**: Removes the entire table structure and data, cannot be rolled back
  ```sql
  DROP TABLE employees;
  ```

---

## SELECT Statements

### Basic SELECT
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary FROM employees;

-- Select with alias
SELECT first_name AS "First Name", salary * 12 AS "Annual Salary"
FROM employees;
```

### DISTINCT
```sql
-- Get unique departments
SELECT DISTINCT department FROM employees;
```

### Interview Question 2: What is the difference between WHERE and HAVING?

**Answer:**
- **WHERE**: Filters rows BEFORE grouping, cannot use aggregate functions
- **HAVING**: Filters groups AFTER grouping, can use aggregate functions

```sql
-- WHERE example
SELECT department, AVG(salary)
FROM employees
WHERE salary > 50000  -- Filter before grouping
GROUP BY department;

-- HAVING example
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000;  -- Filter after grouping
```

---

## WHERE Clause & Filtering

### Comparison Operators
```sql
-- Equal, Not Equal
SELECT * FROM employees WHERE department = 'IT';
SELECT * FROM employees WHERE department != 'HR';

-- Greater than, Less than
SELECT * FROM employees WHERE salary > 70000;
SELECT * FROM employees WHERE hire_date < '2020-01-01';

-- BETWEEN
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 80000;

-- IN
SELECT * FROM employees WHERE department IN ('IT', 'Sales', 'Marketing');

-- LIKE (Pattern matching)
SELECT * FROM employees WHERE email LIKE '%@gmail.com';
SELECT * FROM employees WHERE first_name LIKE 'J%';  -- Starts with J
SELECT * FROM employees WHERE last_name LIKE '%son';  -- Ends with son
SELECT * FROM employees WHERE first_name LIKE '_ohn';  -- Second char is 'o'

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE email IS NULL;
SELECT * FROM employees WHERE department IS NOT NULL;
```

### Logical Operators
```sql
-- AND
SELECT * FROM employees 
WHERE department = 'IT' AND salary > 60000;

-- OR
SELECT * FROM employees 
WHERE department = 'IT' OR department = 'Sales';

-- NOT
SELECT * FROM employees 
WHERE NOT department = 'HR';
```

---

## JOIN Operations

### Sample Tables for JOIN Examples
```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    salary DECIMAL(10, 2)
);
```

### INNER JOIN
Returns only matching rows from both tables.
```sql
SELECT e.emp_name, e.salary, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### LEFT JOIN (LEFT OUTER JOIN)
Returns all rows from left table and matching rows from right table.
```sql
SELECT e.emp_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### RIGHT JOIN (RIGHT OUTER JOIN)
Returns all rows from right table and matching rows from left table.
```sql
SELECT e.emp_name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

### FULL OUTER JOIN
Returns all rows when there's a match in either table.
```sql
SELECT e.emp_name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

### CROSS JOIN
Returns Cartesian product of both tables.
```sql
SELECT e.emp_name, d.dept_name
FROM employees e
CROSS JOIN departments d;
```

### SELF JOIN
Join a table to itself.
```sql
-- Find employees with same salary
SELECT e1.emp_name AS Employee1, e2.emp_name AS Employee2, e1.salary
FROM employees e1
INNER JOIN employees e2 ON e1.salary = e2.salary AND e1.emp_id < e2.emp_id;
```

### Interview Question 3: Explain the difference between INNER JOIN and LEFT JOIN with an example.

**Answer:**
**INNER JOIN** returns only the rows where there is a match in both tables.
**LEFT JOIN** returns all rows from the left table and matching rows from the right table. If no match, NULL values are returned for right table columns.

Example:
```sql
-- Sample Data
-- employees: (1, 'John', 10), (2, 'Jane', 20), (3, 'Bob', NULL)
-- departments: (10, 'IT'), (20, 'Sales')

-- INNER JOIN - Returns only John and Jane (Bob has no department)
SELECT e.emp_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
-- Result: John-IT, Jane-Sales

-- LEFT JOIN - Returns all employees including Bob
SELECT e.emp_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
-- Result: John-IT, Jane-Sales, Bob-NULL
```

---

## Aggregate Functions

### Common Aggregate Functions
```sql
-- COUNT
SELECT COUNT(*) FROM employees;
SELECT COUNT(DISTINCT department) FROM employees;

-- SUM
SELECT SUM(salary) AS total_salary FROM employees;

-- AVG
SELECT AVG(salary) AS average_salary FROM employees;

-- MIN and MAX
SELECT MIN(salary) AS lowest_salary FROM employees;
SELECT MAX(salary) AS highest_salary FROM employees;

-- Multiple aggregates
SELECT 
    COUNT(*) AS total_employees,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees;
```

---

## GROUP BY & HAVING

### GROUP BY
Groups rows with same values into summary rows.
```sql
-- Count employees by department
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- Average salary by department
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;

-- Multiple columns in GROUP BY
SELECT department, YEAR(hire_date) AS hire_year, COUNT(*) AS count
FROM employees
GROUP BY department, YEAR(hire_date);
```

### HAVING
Filters groups after aggregation.
```sql
-- Departments with more than 5 employees
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- Departments with average salary > 60000
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000
ORDER BY avg_salary DESC;
```

### Interview Question 4: Write a query to find the second highest salary.

**Answer:**
```sql
-- Method 1: Using LIMIT/OFFSET
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Method 2: Using subquery
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method 3: Using DENSE_RANK (Window Function)
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) AS ranked
WHERE rank = 2;
```

---

## Subqueries

### Subquery in WHERE Clause
```sql
-- Employees earning more than average
SELECT emp_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Employees in IT department
SELECT emp_name
FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE dept_name = 'IT');
```

### Subquery in FROM Clause (Derived Table)
```sql
SELECT dept_name, avg_salary
FROM (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
) AS dept_avg
JOIN departments d ON dept_avg.dept_id = d.dept_id
WHERE avg_salary > 60000;
```

### Correlated Subquery
Subquery that references columns from outer query.
```sql
-- Employees earning more than their department average
SELECT e1.emp_name, e1.salary, e1.dept_id
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
);
```

### EXISTS
```sql
-- Departments that have employees
SELECT dept_name
FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.dept_id = d.dept_id
);
```

### Interview Question 5: What's the difference between IN and EXISTS?

**Answer:**
- **IN**: Compares a value against a list of values, subquery executes first
- **EXISTS**: Checks if subquery returns any rows, stops as soon as match is found (more efficient for large datasets)

```sql
-- IN - Subquery returns all dept_ids, then checks each employee
SELECT emp_name
FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE dept_name = 'IT');

-- EXISTS - Stops as soon as match is found (better performance)
SELECT emp_name
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d 
    WHERE d.dept_id = e.dept_id AND d.dept_name = 'IT'
);
```

**Performance**: EXISTS is generally faster for large datasets because it uses short-circuit evaluation.

---

## Window Functions

Window functions perform calculations across a set of rows related to the current row.

### ROW_NUMBER()
Assigns unique sequential number to each row.
```sql
SELECT 
    emp_name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;
```

### RANK() and DENSE_RANK()
```sql
SELECT 
    emp_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- RANK: 1, 2, 2, 4 (skips 3)
-- DENSE_RANK: 1, 2, 2, 3 (no skip)
```

### PARTITION BY
Divides result set into partitions.
```sql
-- Rank employees within each department
SELECT 
    emp_name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

### LAG() and LEAD()
Access data from previous or next row.
```sql
-- Compare each employee's salary with previous employee
SELECT 
    emp_name,
    salary,
    LAG(salary, 1) OVER (ORDER BY salary) AS prev_salary,
    LEAD(salary, 1) OVER (ORDER BY salary) AS next_salary
FROM employees;
```

### NTILE()
Distributes rows into specified number of groups.
```sql
-- Divide employees into 4 salary quartiles
SELECT 
    emp_name,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;
```

### Interview Question 6: Find the top 3 highest paid employees in each department.

**Answer:**
```sql
SELECT emp_name, department, salary
FROM (
    SELECT 
        emp_name,
        department,
        salary,
        DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
) AS ranked
WHERE rank <= 3;
```

---

## Indexes

### What is an Index?
An index is a database object that improves the speed of data retrieval operations on a table.

### Creating Indexes
```sql
-- Single column index
CREATE INDEX idx_employee_name ON employees(emp_name);

-- Composite index (multiple columns)
CREATE INDEX idx_dept_salary ON employees(dept_id, salary);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Drop index
DROP INDEX idx_employee_name;
```

### Interview Question 7: What are the advantages and disadvantages of indexes?

**Answer:**

**Advantages:**
- Faster SELECT queries and WHERE clause filtering
- Speeds up JOIN operations
- Improves ORDER BY and GROUP BY performance
- Enforces uniqueness (UNIQUE indexes)

**Disadvantages:**
- Slows down INSERT, UPDATE, DELETE operations (index must be updated)
- Requires additional disk space
- Too many indexes can degrade performance
- Maintenance overhead

**When to use indexes:**
- Columns frequently used in WHERE clauses
- Columns used in JOIN conditions
- Columns used in ORDER BY
- Foreign key columns

**When NOT to use indexes:**
- Small tables
- Columns with frequent INSERT/UPDATE/DELETE
- Columns with low cardinality (few distinct values like gender)

---

## Transactions & ACID

### What is a Transaction?
A transaction is a sequence of operations performed as a single logical unit of work.

### Transaction Commands
```sql
-- Start transaction
BEGIN TRANSACTION;

-- Perform operations
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Commit (save changes)
COMMIT;

-- Or Rollback (undo changes)
ROLLBACK;

-- Savepoint
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 50 WHERE account_id = 2;
ROLLBACK TO sp1;  -- Undo only second update
COMMIT;
```

### Interview Question 8: Explain ACID properties with examples.

**Answer:**

**A - Atomicity**: All operations in a transaction succeed or all fail (all-or-nothing).
```sql
-- Either both updates happen or neither
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;  -- If any fails, both rollback
```

**C - Consistency**: Database moves from one valid state to another, maintaining all rules.
```sql
-- Total money before = Total money after
-- If account 1 has $500 and account 2 has $300 (total $800)
-- After transfer, account 1 has $400 and account 2 has $400 (still $800)
```

**I - Isolation**: Concurrent transactions don't interfere with each other.
```sql
-- Transaction 1 and Transaction 2 run simultaneously
-- Each sees consistent data without seeing partial results of the other
```

**D - Durability**: Once committed, changes are permanent even if system crashes.
```sql
COMMIT;  -- After this, data is saved permanently to disk
```

---

## Normalization

### What is Normalization?
Process of organizing data to reduce redundancy and improve data integrity.

### Normal Forms

**1NF (First Normal Form)**
- Each column contains atomic (indivisible) values
- Each column contains values of single type
- Each column has unique name
- Order doesn't matter

```sql
-- NOT in 1NF (phone_numbers has multiple values)
CREATE TABLE employees_bad (
    emp_id INT,
    emp_name VARCHAR(50),
    phone_numbers VARCHAR(100)  -- "123-456, 789-012"
);

-- In 1NF
CREATE TABLE employees_good (
    emp_id INT,
    emp_name VARCHAR(50),
    phone_number VARCHAR(20)
);
```

**2NF (Second Normal Form)**
- Must be in 1NF
- No partial dependencies (non-key attributes depend on entire primary key)

```sql
-- NOT in 2NF (course_name depends only on course_id, not on both)
CREATE TABLE enrollments_bad (
    student_id INT,
    course_id INT,
    course_name VARCHAR(50),
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id)
);

-- In 2NF (separate tables)
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(50)
);

CREATE TABLE enrollments_good (
    student_id INT,
    course_id INT,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id)
);
```

**3NF (Third Normal Form)**
- Must be in 2NF
- No transitive dependencies (non-key attributes don't depend on other non-key attributes)

```sql
-- NOT in 3NF (dept_name depends on dept_id, not emp_id)
CREATE TABLE employees_bad (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    dept_name VARCHAR(50)
);

-- In 3NF
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees_good (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

### Interview Question 9: What is denormalization and when would you use it?

**Answer:**
**Denormalization** is intentionally introducing redundancy by combining tables to improve read performance.

**When to use:**
- Read-heavy applications where query performance is critical
- Reporting and analytics databases
- When JOIN operations are too expensive
- Data warehousing

**Example:**
```sql
-- Normalized (requires JOIN)
SELECT e.emp_name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- Denormalized (no JOIN needed, but dept_name is duplicated)
CREATE TABLE employees_denorm (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    dept_id INT,
    dept_name VARCHAR(50)  -- Redundant data
);

SELECT emp_name, dept_name FROM employees_denorm;  -- Faster
```

**Trade-offs:**
- Pros: Faster reads, simpler queries
- Cons: Data redundancy, update anomalies, more storage

---

## Common Interview Questions

### Question 10: Find duplicate records in a table.

**Answer:**
```sql
-- Find duplicate emails
SELECT email, COUNT(*) AS count
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;

-- Get all details of duplicate records
SELECT *
FROM employees
WHERE email IN (
    SELECT email
    FROM employees
    GROUP BY email
    HAVING COUNT(*) > 1
);
```

### Question 11: Delete duplicate records keeping one.

**Answer:**
```sql
-- Using ROW_NUMBER()
DELETE FROM employees
WHERE emp_id IN (
    SELECT emp_id
    FROM (
        SELECT emp_id,
               ROW_NUMBER() OVER (PARTITION BY email ORDER BY emp_id) AS rn
        FROM employees
    ) AS temp
    WHERE rn > 1
);

-- Alternative using self-join
DELETE e1
FROM employees e1
INNER JOIN employees e2
WHERE e1.emp_id > e2.emp_id AND e1.email = e2.email;
```

### Question 12: Find employees who earn more than their manager.

**Answer:**
```sql
-- Assuming employees table has manager_id column
SELECT e.emp_name AS employee, e.salary AS emp_salary,
       m.emp_name AS manager, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

### Question 13: Find the Nth highest salary.

**Answer:**
```sql
-- For N = 3 (third highest)
-- Method 1: Using DENSE_RANK
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) AS ranked
WHERE rank = 3;

-- Method 2: Using subquery
SELECT DISTINCT salary
FROM employees e1
WHERE 3 = (
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary >= e1.salary
);

-- Method 3: Using LIMIT OFFSET
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;  -- For 3rd highest (offset is N-1)
```

### Question 14: Display cumulative sum of salaries.

**Answer:**
```sql
SELECT 
    emp_name,
    salary,
    SUM(salary) OVER (ORDER BY emp_id) AS cumulative_salary
FROM employees;
```

### Question 15: Find employees hired in the last 30 days.

**Answer:**
```sql
-- Using CURRENT_DATE
SELECT emp_name, hire_date
FROM employees
WHERE hire_date >= CURRENT_DATE - INTERVAL '30 days';

-- Using DATEADD (SQL Server)
SELECT emp_name, hire_date
FROM employees
WHERE hire_date >= DATEADD(day, -30, GETDATE());

-- Using DATE_SUB (MySQL)
SELECT emp_name, hire_date
FROM employees
WHERE hire_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```

### Question 16: Pivot table - Convert rows to columns.

**Answer:**
```sql
-- Sample data: sales by quarter
-- Original: (Product, Quarter, Sales)
-- Desired: (Product, Q1, Q2, Q3, Q4)

-- Using CASE statements
SELECT 
    product,
    SUM(CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END) AS Q1,
    SUM(CASE WHEN quarter = 'Q2' THEN sales ELSE 0 END) AS Q2,
    SUM(CASE WHEN quarter = 'Q3' THEN sales ELSE 0 END) AS Q3,
    SUM(CASE WHEN quarter = 'Q4' THEN sales ELSE 0 END) AS Q4
FROM sales_data
GROUP BY product;

-- Using PIVOT (SQL Server)
SELECT *
FROM sales_data
PIVOT (
    SUM(sales)
    FOR quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS pivot_table;
```

### Question 17: What's the difference between UNION and UNION ALL?

**Answer:**
- **UNION**: Combines results and removes duplicates (slower)
- **UNION ALL**: Combines results and keeps duplicates (faster)

```sql
-- UNION (removes duplicates)
SELECT emp_name FROM employees WHERE dept_id = 1
UNION
SELECT emp_name FROM employees WHERE salary > 50000;

-- UNION ALL (keeps duplicates)
SELECT emp_name FROM employees WHERE dept_id = 1
UNION ALL
SELECT emp_name FROM employees WHERE salary > 50000;
```

**When to use:**
- Use UNION when you need unique results
- Use UNION ALL when duplicates are acceptable or impossible (better performance)

### Question 18: Explain different types of keys in SQL.

**Answer:**

**Primary Key:**
- Uniquely identifies each record
- Cannot be NULL
- Only one per table
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50)
);
```

**Foreign Key:**
- Links two tables together
- References primary key of another table
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    emp_id INT,
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);
```

**Unique Key:**
- Ensures all values are unique
- Can be NULL (only one NULL allowed)
- Multiple unique keys per table
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
```

**Candidate Key:**
- Column(s) that can qualify as primary key
- Example: emp_id, email, ssn all could be primary keys

**Composite Key:**
- Primary key made of multiple columns
```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id)
);
```

### Question 19: Write a query to find employees with no manager.

**Answer:**
```sql
-- Assuming manager_id column exists
SELECT emp_name
FROM employees
WHERE manager_id IS NULL;

-- Or using NOT EXISTS
SELECT e1.emp_name
FROM employees e1
WHERE NOT EXISTS (
    SELECT 1 FROM employees e2 WHERE e2.emp_id = e1.manager_id
);
```

### Question 20: Calculate running average of salaries.

**Answer:**
```sql
SELECT 
    emp_name,
    salary,
    AVG(salary) OVER (ORDER BY emp_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_avg
FROM employees;

-- Or using simpler syntax
SELECT 
    emp_name,
    salary,
    AVG(salary) OVER (ORDER BY emp_id) AS running_avg
FROM employees;
```

---

## Quick Tips for SQL Interviews

1. **Understand the problem**: Read the question carefully, clarify if needed
2. **Think about edge cases**: NULL values, empty tables, duplicates
3. **Optimize**: Consider indexes, avoid SELECT *, use appropriate JOINs
4. **Explain your approach**: Talk through your solution
5. **Test your query**: Walk through with sample data
6. **Know your database**: Syntax varies (MySQL, PostgreSQL, SQL Server, Oracle)
7. **Practice common patterns**: Window functions, self-joins, subqueries
8. **Understand performance**: Explain time complexity when asked

---

## Practice Exercises

### Exercise 1: Employee Database
Given tables `employees(emp_id, emp_name, dept_id, salary, manager_id)` and `departments(dept_id, dept_name)`:

1. Find departments with average salary > 60000
2. List employees earning more than their department average
3. Find the department with the highest total salary
4. Display employee hierarchy (employee -> manager -> manager's manager)

### Exercise 2: Sales Database
Given `orders(order_id, customer_id, order_date, amount)` and `customers(customer_id, customer_name, city)`:

1. Find top 5 customers by total purchase amount
2. Calculate month-over-month sales growth
3. Find customers who haven't ordered in the last 90 days
4. Display running total of sales by date

---

## Conclusion

This tutorial covers the essential SQL concepts for interview preparation. Practice these queries, understand the underlying concepts, and you'll be well-prepared for your SQL interview!

**Key Takeaways:**
- Master JOINs and understand when to use each type
- Be comfortable with aggregate functions and GROUP BY
- Understand window functions for advanced analytics
- Know the difference between WHERE and HAVING
- Practice subqueries and correlated subqueries
- Understand indexes and their impact on performance
- Know ACID properties and transaction management
- Be familiar with normalization concepts

Good luck with your interview! 🚀
