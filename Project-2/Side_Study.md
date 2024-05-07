# LEMP Self Side Study

## SQL Syntax and Most Common Commands

### Introduction
SQL (Structured Query Language) is a powerful tool used for managing and manipulating relational databases and this documentation aims to provide a clear overview of SQL syntax and the most frequently used commands.

### SQL Syntax Basics

SQL queries are written to retrieve, manipulate, and manage data stored in a relational database. The basic syntax for an SQL query follows a structured format:

```
SELECT column1, column2, column3,
FROM table_name
WHERE condition
```
- **SELECT**: Specifies which columns you want to retrieve data from.
- **FROM**: Specifies the table from which to retrieve the data.
- **WHERE**: Optional clause used to filter records based on specified conditions.

### Commonly Used SQL Commands

- **SELECT Statement**

The SELECT statement is used to retrieve data from one or more tables.

Example:

```
SELECT first_name, last_name
FROM employees;
```

**Explanation**:

*SELECT*: Specifies the columns you want to retrieve data from.

*FROM*: Specifies the table from which to retrieve the data.


- **INSERT Statement**

The INSERT statement is used to add new records to a table.

Example:

```
INSERT INTO employees (first_name, last_name, age)
VALUES ('John', 'Doe', 30);
```

**Explanation**:

*INSERT INTO*: Specifies the table to which you want to add new records.

*VALUES*: Specifies the values you want to insert into the corresponding columns.

- **UPDATE Statement**

The UPDATE statement is used to modify existing records in a table.

Example:

```
UPDATE employees
SET age = 31
WHERE last_name = 'Doe';
```

**Explanation**:

*UPDATE*: Specifies the table in which you want to update records.

*SET*: Specifies the columns you want to update and their new values.

*WHERE*: Optional clause used to specify which records to update based on certain conditions.


- **DELETE Statement**

The DELETE statement is used to remove records from a table.

Example:

```
DELETE FROM employees
WHERE last_name = 'Doe';
```

**Explanation**:

*DELETE FROM*: Specifies the table from which you want to delete records.

*WHERE*: Optional clause used to specify which records to delete based on certain conditions.

- **CREATE DATABASE Statement**

The CREATE DATABASE statement is used to create a new database.

Example:
```
CREATE DATABASE mydatabase;
```
**Explanation**

*mydatabase*: Name of the new database to be created.

- **ALTER DATABASE Statement**

The ALTER DATABASE statement is used to modify a database.

Example:

```
ALTER DATABASE mydatabase;
```

**Explanation**

*mydatabase*: Name of the database to be modified.

- **CREATE TABLE Statement**

The CREATE TABLE statement is used to create a new table.

Example:

```
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    age INT
);
```

**Explanation**

*INT, VARCHAR(50)*: Datatypes of the columns.

*id*: An integer column designated as the primary key.

*PRIMARY KEY*: Defines a primary key for the table, which uniquely identifies each row.

*AUTO_INCREMENT*: Automatically increments the value of the "id" column for each new row added to the table.

- **ALTER TABLE Statement**

The ALTER TABLE statement is used to modify a table by adding a new column.

Example:

```
ALTER TABLE employees
ADD email VARCHAR(100);
```

**Explanation**

*employees*: Name of the table to be modified.

*email*: Name of the new column to be added.

*VARCHAR(100)*: Datatype of the new column.

- **DROP TABLE Statement**

The DROP TABLE statement is used to delete a table.

Example:

```
DROP TABLE employees;
```
**Explanation**

*employees*: Name of the table to be deleted.


- **CREATE INDEX Statement**

The CREATE INDEX statement is used to create an index (search key).

Example:

```
CREATE INDEX idx_last_name
ON employees (last_name);
```
**Explanation**

*idx_last_name*: Name of the index to be created.

*last_name*: Column on which the index is created.

- **DROP INDEX Statement**

The DROP INDEX statement is used to delete an index.

Example:

```
DROP INDEX idx_last_name;
```

**Explanation**

*idx_last_name*: Name of the index to be deleted.

#### N.B:

- SQL keywords are not case sensitive, **select** is the same as **SELECT**.
- Semicolon (;) is the standard way to separate each SQL statement in database systems that allow more than one SQL statement to be executed in the same call to the server.
- In SQL, (*) is a wildcard character that represents "all columns" in a table. It selects all columns from the specified table in a query.

### Conclusion

Understanding the basic SQL syntax and commonly used commands is essential for effectively querying and managing data in relational databases. By mastering these concepts, you'll be well-equipped to work with databases and perform a wide range of data manipulation tasks.


## Nano Editor

Nano is a straightforward text editor that's often preferred by system adminstrators for its simplicity and ease of use. Here's a step-by-step guide to help you get comfortable using nano.

1. **Opening a File**:
- To open an existing file with nano, simply type nano filename in your terminal and press Enter. Replace "filename" with the name of the file you want to open.
  > nano filename
- If the file doesn't exist, nano will create a new one with that name when you save it.

2. **Basic Navigation**:
- Nano provides easy-to-use keyboard shortcuts for navigation. The arrow keys will move the cursor around the text.
- Alternatively, you can use the following keyboard shortcuts:
  - Move cursor up: ↑
  - Move cursor down: ↓
  - Move cursor left: ←
  - Move cursor right: →
  - You can also use Ctrl + P (previous) and Ctrl + N (next) for up and down navigation.

3. **Editing Text**:
- To start typing, simply place the cursor where you want to insert text and start typing.
- Use the Backspace key to delete characters to the left of the cursor.
- Use the Delete key to delete characters to the right of the cursor.
- Nano also supports basic text editing shortcuts:
  - Cut **(Ctrl + K)**: Deletes the line from the cursor to the end of the line and saves it to the clipboard.
  - Paste **(Ctrl + U)**: Pastes the previously cut text at the cursor position.
  - Undo **(Ctrl + U)**: Undo the last action.
  - Redo **(Alt + U)**: Redo the last undone action.

4. **Saving and Exiting**:
- To save your changes and exit nano, press **Ctrl + O** (Write Out) and then Enter.
- If you want to save changes to an existing file without exiting, you can just press **Ctrl + O** and **Enter**.
- To exit nano without saving changes, press **Ctrl + X** (Exit). If there are unsaved changes, nano will prompt you to save before exiting.

5. **Searching**:
- Nano allows you to search for text within the file. Press **Ctrl + W**, type your search term, and press **Enter**. nano will jump to the first occurrence of the term. Press **Ctrl + W** again to find the next occurrence.

6. **Help**:
- If you ever forget a command or need help, you can access nano's built-in help system by pressing **Ctrl + G** (To Get Help). This will display a list of common commands at the bottom of the screen.

7. **Customization**:
- Nano allows some basic customization through its configuration file **(~/.nanorc)**. You can adjust settings such as syntax highlighting, key bindings, and more to suit your preferences.

---