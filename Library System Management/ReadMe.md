# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/AdhavanHero/MY-SQL-Projects/blob/main/Library%20System%20Management/huge-library%207th%20pricnce.png)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/AdhavanHero/MY-SQL-Projects/blob/main/Library%20System%20Management/ERD.PNG)

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: The database includes six core tables:
branch
employees
members
books
issued_status
return_status
Each table has been normalized and linked through foreign keys for data integrity.

```sql
CREATE DATABASE library_db;

-- Table: Branch
CREATE TABLE branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

-- Table: Employees
CREATE TABLE employees (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

-- Table: Members
CREATE TABLE members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Table: Books
CREATE TABLE books (
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(80),
    category VARCHAR(30),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),
    author VARCHAR(30),
    publisher VARCHAR(30)
);

-- Table: Issued Status
CREATE TABLE issued_status (
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

-- Table: Return Status
CREATE TABLE return_status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```

and importing the datasets accoding to the table manually

### ✏️ 2. CRUD Operations
🟢 Task 1 – Create a New Book Record
Insert a new book titled To Kill a Mockingbird into the books table.
```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

🟡 Task 2 – Update Member Address
Update the address of member C103.
```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

🔴 Task 3 – Delete Issued Record
Remove an issued record by issued_id.

DELETE FROM issued_status
WHERE issued_id = 'IS121';

🔍 Task 4 – Retrieve Books Issued by a Specific Employee
Find all books issued by employee E101.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
```

👥 Task 5 – List Members Who Issued More Than One Book
Group by employee ID to find multiple issuances.
```sql
SELECT issued_emp_id, COUNT(*) AS total_issued
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(*) > 1;
```
### 🧱 3. CTAS (Create Table As Select)

🧾 Task 6 – Create Book Issue Summary
Generate a table showing how many times each book was issued.
```sql
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status AS ist
JOIN books AS b ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
```

Task 7 – Retrieve All Books by Category
```sql
SELECT * FROM books
WHERE category = 'Classic';
```

Task 8 – Total Rental Income by Category
```sql
SELECT 
    b.category,
    SUM(b.rental_price) AS total_income,
    COUNT(*) AS total_books
FROM issued_status AS ist
JOIN books AS b
ON b.isbn = ist.issued_book_isbn
GROUP BY b.category;
```

Task 9 – Members Registered in Last 180 Days
```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

Task 10 – Employee Details with Manager and Branch Info
```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.branch_id,
    b.branch_address,
    e2.emp_name AS manager
FROM employees AS e1
JOIN branch AS b ON e1.branch_id = b.branch_id
JOIN employees AS e2 ON e2.emp_id = b.manager_id;
```

Task 11 – Books with High Rental Price
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

Task 12 – Books Not Yet Returned
```sql
SELECT * FROM issued_status AS ist
LEFT JOIN return_status AS rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

### 🧠 4. Advanced SQL Operations
```sql
Task 13 – Identify Members with Overdue Books
(Assuming 30-day return period)

SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date AS overdue_days
FROM issued_status AS ist
JOIN members AS m ON m.member_id = ist.issued_member_id
JOIN books AS bk ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL
  AND (CURRENT_DATE - ist.issued_date) > 30
ORDER BY overdue_days DESC;
```

Task 14 – Procedure: Update Book Status When Returned
A stored procedure to automatically mark returned books as “yes”.
```sql
CREATE OR REPLACE PROCEDURE add_return_records(
    p_return_id VARCHAR(10),
    p_issued_id VARCHAR(10),
    p_book_quality VARCHAR(10)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
BEGIN
    INSERT INTO return_status(return_id, issued_id, return_date, return_book_isbn)
    VALUES (p_return_id, p_issued_id, CURRENT_DATE, v_isbn);

    SELECT issued_book_isbn, issued_book_name INTO v_isbn, v_book_name
    FROM issued_status WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Book returned successfully: %', v_book_name;
END;
$$;

#procedure test

CALL add_return_records('RS138', 'IS135', 'Good');
```

Task 15 – Branch Performance Report
Generate summary reports for each branch.
```sql
CREATE TABLE branch_reports AS
SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) AS number_book_issued,
    COUNT(rs.return_id) AS number_book_return,
    SUM(bk.rental_price) AS total_revenue
FROM issued_status AS ist
JOIN employees AS e ON e.emp_id = ist.issued_emp_id
JOIN branch AS b ON e.branch_id = b.branch_id
LEFT JOIN return_status AS rs ON rs.issued_id = ist.issued_id
JOIN books AS bk ON ist.issued_book_isbn = bk.isbn
GROUP BY b.branch_id, b.manager_id;
```

Task 16 – CTAS: Create Table of Active Members
Identify members who issued at least one book in the last 2 months.
```sql
CREATE TABLE active_members AS
SELECT DISTINCT m.member_id, m.member_name, m.reg_date
FROM issued_status AS ist
JOIN members AS m ON ist.issued_member_id = m.member_id
WHERE ist.issued_date >= CURRENT_DATE - INTERVAL '60 days';
```

##🧾 Final Notes
###✅ Skills Demonstrated:

Database design & schema normalization
CRUD and CTAS operations
Stored procedures in PL/pgSQL
Analytical and join-based SQL queries

###✅ Key Takeaways:
This project demonstrates practical SQL knowledge to manage a library efficiently—tracking members, books, staff, and branch performance through meaningful data analysis.

