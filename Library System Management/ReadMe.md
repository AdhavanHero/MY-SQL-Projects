# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/najirh/Library-System-Management---P2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD]([https://github.com/najirh/Library-System-Management---P2/blob/main/library_erd.png](https://github.com/AdhavanHero/MY-SQL-Projects/blob/main/Library%20System%20Management/ERD.png)

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


```

