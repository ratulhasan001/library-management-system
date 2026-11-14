# Library Management System

A desktop library management application written in Java with a Swing GUI (designed in the NetBeans GUI builder) and a MySQL database accessed through JDBC. A librarian registers and signs in, then uses an admin panel to manage the catalogue (categories, authors, publishers, books), library members, and book issue, renewal, return and reservation, including an overdue fine on return. It was built as a team project for the course CSE-2218 Advanced Programming Sessional at BAUET (2024).

Project report: [Ratul_Final_Project_Report_LMS.pdf](Ratul_Final_Project_Report_LMS.pdf) (also available as [.docx](Ratul_Final_Project_Report_LMS.docx)). It includes the use case, sequence, data flow and ER diagrams and screenshots of every screen.

## Features

- **Sign up / sign in** for library staff (name, e-mail, designation, password); duplicate e-mails are rejected
- **Admin panel** linking to every module
- **Categories** with an Active/Deactive status; add, edit, delete
- **Authors** and **Publishers**: add, edit, delete
- **Books**: add, edit, delete; category, author and publisher are chosen from drop-downs filled from the database
- **Members** (ID, name, department, batch, e-mail, mobile): add, edit, delete
- **Issue book**: look up a member by ID, pick a book (only books in Active categories are listed) and choose issue and return dates with a date picker
- **Renew book**: look up a member's loan by member ID and update its issue and return dates
- **Return book**: shows the due date, days overdue and a fine of 50 per overdue day, then removes the loan
- **Reserve book**: record a reservation with a pick-up date
- **Search book**: find books matching a chosen category, author, publisher and edition
- **Library ID card**: fill in member details, generate a card preview and print it

## Tech stack

- Java 17 (`javac.source`/`javac.target` = 17 in `nbproject/project.properties`), Swing/AWT
- NetBeans IDE project (Ant build: `build.xml`, `nbproject/`)
- MySQL via MySQL Connector/J 8.0.28 (`mysql-connector-java-8.0.28.jar`)
- JCalendar 1.4 (`jcalendar-1.4.jar`) for date pickers

## Project structure

```
library-management-system/
├── src/
│   ├── signin.java          # Main class (login window)
│   ├── registration.java    # Staff sign-up
│   ├── mainmenu.java        # Admin panel
│   ├── category.java  author.java  Publisher.java  Book.java  Member.java
│   ├── Lend.java            # Issue book
│   ├── Renew.java  Return.java  Reserve.java  Search.java  idcard.java
│   ├── dbConnection.java    # JDBC connection settings
│   ├── *.form               # NetBeans GUI builder layouts
│   └── image/               # Icons and images used by the UI
├── dist/
│   ├── LibraryMS.jar        # Pre-built runnable jar (Main-Class: signin)
│   └── lib/                 # Dependency jars referenced by LibraryMS.jar
├── nbproject/               # NetBeans project configuration
├── build.xml
├── jcalendar-1.4.jar
├── mysql-connector-java-8.0.28.jar
└── Ratul_Final_Project_Report_LMS.pdf / .docx
```

## Getting started

### Prerequisites

- JDK 17 or newer (the compiled classes target Java 17)
- MySQL server running on `localhost:3306` (for example via XAMPP)
- NetBeans IDE, only if you want to edit or rebuild the project

### 1. Clone

```bash
git clone https://github.com/ratulhasan001/library-management-system.git
cd library-management-system
```

### 2. Create the database

`src/dbConnection.java` connects with these hard-coded settings:

| Setting  | Value |
| -------- | ----- |
| URL      | `jdbc:mysql://localhost:3306/library` |
| User     | `root` |
| Password | *(empty)*, the XAMPP default |

To use other credentials, edit `dbConnection.java` and rebuild.

The repository does **not** include a SQL schema or dump, so the `library` database has to be created first. The script below is reconstructed from the SQL statements in the source code: table and column names match what the code uses, but the column types and keys are assumptions, not the original schema.

```sql
CREATE DATABASE library;
USE library;

CREATE TABLE user (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  name        VARCHAR(100),
  email       VARCHAR(100) UNIQUE,
  Designation VARCHAR(100),
  password    VARCHAR(100)
);

CREATE TABLE category (
  id       INT AUTO_INCREMENT PRIMARY KEY,
  category VARCHAR(100),
  status   VARCHAR(20)            -- 'Active' or 'Deactive'
);

CREATE TABLE author (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE publisher (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100)
);

-- category, author and publisher are stored by name, not by id
CREATE TABLE book (
  id        INT AUTO_INCREMENT PRIMARY KEY,
  name      VARCHAR(200),
  category  VARCHAR(100),
  author    VARCHAR(100),
  publisher VARCHAR(100),
  edition   VARCHAR(50)
);

-- ID is entered manually on the Member screen
CREATE TABLE member (
  ID         INT PRIMARY KEY,
  Name       VARCHAR(100),
  Department VARCHAR(50),
  Batch      VARCHAR(20),
  Email      VARCHAR(100),
  Mobile     VARCHAR(20)
);

-- issued books
CREATE TABLE ibook (
  id    INT AUTO_INCREMENT PRIMARY KEY,
  mid   INT,                      -- member ID
  mname VARCHAR(100),             -- member name
  book  VARCHAR(200),             -- book name
  idate DATE,                     -- issue date
  rdate DATE                      -- return (due) date
);

-- reserved books
CREATE TABLE rbook (
  id        INT AUTO_INCREMENT PRIMARY KEY,
  mid       INT,
  mname     VARCHAR(100),
  book      VARCHAR(200),
  category  VARCHAR(100),
  author    VARCHAR(100),
  publisher VARCHAR(100),
  edition   VARCHAR(50),
  pickdate  DATE
);
```

Save it as e.g. `schema.sql` and load it with `mysql -u root < schema.sql`, or paste it into phpMyAdmin.

### 3. Run

Using the pre-built jar (it loads its dependencies from `dist/lib/`):

```bash
java -jar dist/LibraryMS.jar
```

Or open the folder in NetBeans (**File > Open Project**), then **Clean and Build** and **Run**. The build regenerates `build/` and `dist/`.

On first launch, use **Sign Up** to create a staff account, then sign in.

## Limitations

- The edit actions on the Book, Member, Issue Book and Reserve Book screens run `UPDATE` statements without a `WHERE` clause, so saving an edit there overwrites every row in that table. (Edits on the Category, Author, Publisher and Renew screens are correctly limited to one record.)
- Passwords are stored and compared in plain text, and the database credentials are hard-coded.
- Returning a book deletes all loans recorded for that member ID.
- No SQL schema is shipped with the project (see step 2).

## Authors

Team project by Maisha Tasmim, Yousuf Oley and Ratul Hasan (Department of CSE, BAUET), as listed in the project report.

---

Ratul Hasan · [ratulhasan.dev](https://ratulhasan.dev)
