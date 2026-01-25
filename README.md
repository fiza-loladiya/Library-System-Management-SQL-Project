# Library-System-Management-SQL-Project
## Project Overview
**Project Title:** Library Management System  
**Level:** Intermediate  
**Database:** `library_db`

This project implements a **Library Management System** using SQL. It covers **database design, table relationships, CRUD operations, CTAS (Create Table As Select)** and **advanced queries** for reporting and analysis.

---

## Objectives
- **Set up Library Database:** Create and populate tables for branches, employees, members, books, issued status, and return status.
- **CRUD Operations:** Insert, update, delete, and retrieve data across tables.
- **CTAS Reporting:** Generate summary/report tables using `CREATE TABLE AS SELECT`.
- **Advanced SQL Analysis:** Write complex queries for business-style insights (issued vs returned, overdue books, branch performance, active members, etc.)

---

## ERD / Data Model
![Library ERD](assets/library_erd.png)

---

## Database Setup

### Create Database
```sql
CREATE DATABASE library_db;
Tables (Schema)
-- Branch
DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

-- Employees
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

-- Members
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Books
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(80),
    category VARCHAR(30),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),  -- yes/no
    author VARCHAR(30),
    publisher VARCHAR(30)
);

-- Issued Status
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(30),
    issued_book_name VARCHAR(80),
    issued_date DATE,
    issued_book_isbn VARCHAR(50),
    issued_emp_id VARCHAR(10),
    FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
    FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn)
);

-- Return Status
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
CRUD Operations (Sample Tasks)

Task 1: Insert a New Book Record

INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

Task 2: Update Member Address

UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';

Task 3: Delete a Record from issued_status

DELETE FROM issued_status
WHERE issued_id = 'IS121';

Task 4: Retrieve All Books Issued by a Specific Employee

SELECT *
FROM issued_status
WHERE issued_emp_id = 'E101';

Task 5: Members Who Issued More Than One Book

SELECT
    issued_member_id,
    COUNT(*) AS total_issues
FROM issued_status
GROUP BY issued_member_id
HAVING COUNT(*) > 1;

CTAS (Create Table As Select)

Task 6: Create Summary Table (Book-wise Issue Count)

CREATE TABLE book_issued_cnt AS
SELECT
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) AS issue_count
FROM issued_status AS ist
JOIN books AS b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;

Data Analysis & Findings (Queries)

Task 7: Books in a Specific Category

SELECT *
FROM books
WHERE category = 'Classic';

Task 8: Total Rental Income by Category

SELECT
    b.category,
    SUM(b.rental_price) AS total_rental_income,
    COUNT(*) AS total_issues
FROM issued_status AS ist
JOIN books AS b
ON b.isbn = ist.issued_book_isbn
GROUP BY b.category;

Task 9: Members Registered in the Last 180 Days

SELECT *
FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';

Task 10: Employees with Branch + Branch Manager Details

SELECT
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.branch_id,
    b.manager_id,
    b.branch_address,
    b.contact_no,
    e2.emp_name AS manager_name
FROM employees AS e1
JOIN branch AS b
ON e1.branch_id = b.branch_id
JOIN employees AS e2
ON e2.emp_id = b.manager_id;

Task 11: Books with Rental Price Above Threshold

CREATE TABLE expensive_books AS
SELECT *
FROM books
WHERE rental_price > 7.00;

Task 12: Books Not Yet Returned

SELECT
    ist.*
FROM issued_status AS ist
LEFT JOIN return_status AS rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;

Advanced SQL

Task 13: Identify Members with Overdue Books (30-day policy)

SELECT
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    (CURRENT_DATE - ist.issued_date) AS overdue_days
FROM issued_status AS ist
JOIN members AS m
ON m.member_id = ist.issued_member_id
JOIN books AS bk
ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL
  AND (CURRENT_DATE - ist.issued_date) > 30
ORDER BY overdue_days DESC;

Task 14: Branch Performance Report (CTAS)

CREATE TABLE branch_reports AS
SELECT
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) AS number_book_issued,
    COUNT(rs.return_id) AS number_of_book_return,
    SUM(bk.rental_price) AS total_revenue
FROM issued_status AS ist
JOIN employees AS e
ON e.emp_id = ist.issued_emp_id
JOIN branch AS b
ON e.branch_id = b.branch_id
LEFT JOIN return_status AS rs
ON rs.issued_id = ist.issued_id
JOIN books AS bk
ON ist.issued_book_isbn = bk.isbn
GROUP BY b.branch_id, b.manager_id;

Task 15: Active Members (Last 2 Months) using CTAS
CREATE TABLE active_members AS
SELECT *
FROM members
WHERE member_id IN (
    SELECT DISTINCT issued_member_id
    FROM issued_status
    WHERE issued_date >= CURRENT_DATE - INTERVAL '2 month'
);

Task 16: Top 3 Employees with Most Book Issues Processed

SELECT
    e.emp_name,
    b.branch_id,
    b.branch_address,
    COUNT(ist.issued_id) AS books_processed
FROM issued_status AS ist
JOIN employees AS e
ON e.emp_id = ist.issued_emp_id
JOIN branch AS b
ON e.branch_id = b.branch_id
GROUP BY e.emp_name, b.branch_id, b.branch_address
ORDER BY books_processed DESC
LIMIT 3;

```
**Conclusion**
This project showcases practical SQL skills by building a complete **Library Management System** from scratch. It covers **database creation, table design with relationships, and day-to-day data operations (CRUD)**, along with **CTAS-based reporting** and **advanced queries** for insights like overdue tracking, active members, and branch-level performance. Overall, it provides a strong foundation in **relational database management** and **analysis-driven querying**, similar to what’s used in real-world systems.


ORDER BY books_processed DESC
LIMIT 3;
