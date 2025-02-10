# Progressive Guide to SQLite Fundamentals with Python

This guide is designed for developers who have Python installed and want to use **SQLite** through Python. The goal is to become familiar with how to interact with SQLite using Python's built-in `sqlite3` module.

---

## 1. Introduction to SQLite in Python

### 1.1 What is SQLite?
- **SQLite** is a lightweight, self-contained relational database.
- It does not require a separate server process.
- Stores data in a single file (`.db` file).
- Fully ACID-compliant with transactional support.

### 1.2 Why Use SQLite with Python?
- **No external dependencies**: SQLite comes built-in with Python.
- **Lightweight and easy to use**.
- **Perfect for small to medium-sized applications**.

### 1.3 Installing SQLite
SQLite comes pre-installed with Python (version 3.7+). To check:
```sh
python -c "import sqlite3; print(sqlite3.version)"
```

---

## 2. DATABASE Operations

### 2.1 Creating and Connecting to a Database
```python
import sqlite3

# Connect to (or create) a database
connection = sqlite3.connect('my_database.db')

# Close the connection
connection.close()
```

### 2.2 Using an In-Memory Database
```python
connection = sqlite3.connect(':memory:')
```

### 2.3 SHOW DATABASES
SQLite does not have a specific SQL command to show databases, as it is a file-based database. Instead, you can check your current connection's file path or list your `.db` files manually.

### 2.4 DROP DATABASE
SQLite does not support the `DROP DATABASE` command. To "delete" a database, simply delete the `.db` file from your filesystem:
```python
import os
os.remove('my_database.db')
```

---

## 3. TABLE Operations

### 3.1 Creating a Table
```python
import sqlite3

connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()

cursor.execute('''
    CREATE TABLE IF NOT EXISTS Customers (
        ID INTEGER PRIMARY KEY AUTOINCREMENT,
        Name TEXT NOT NULL,
        Email TEXT UNIQUE NOT NULL,
        RegistrationDate TEXT DEFAULT CURRENT_TIMESTAMP
    )
''')

connection.commit()
connection.close()
```

### 3.2 SHOW TABLES
SQLite does not support `SHOW TABLES` directly. However, you can query the `sqlite_master` table to get a list of tables:
```python
cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
tables = cursor.fetchall()
for table in tables:
    print(table[0])
```

### 3.3 DROP TABLE
```python
cursor.execute("DROP TABLE IF EXISTS Customers")
```

---

## 4. COLUMN Operations

### 4.1 ADD COLUMN
```python
cursor.execute("ALTER TABLE Customers ADD COLUMN Age INTEGER;")
```

### 4.2 SHOW COLUMNS
SQLite does not have a specific command to show columns. You can query the `PRAGMA table_info()` function to retrieve column details:
```python
cursor.execute("PRAGMA table_info(Customers);")
columns = cursor.fetchall()
for column in columns:
    print(column)
```

### 4.3 DROP COLUMN
SQLite does not directly support dropping columns. To remove a column, you need to:
1. Create a new table without the unwanted column.
2. Copy the data from the old table.
3. Drop the old table.

Example:
```python
# Step 1: Create a new table without the 'Age' column
cursor.execute('''
    CREATE TABLE IF NOT EXISTS NewCustomers (
        ID INTEGER PRIMARY KEY AUTOINCREMENT,
        Name TEXT NOT NULL,
        Email TEXT UNIQUE NOT NULL,
        RegistrationDate TEXT DEFAULT CURRENT_TIMESTAMP
    )
''')

# Step 2: Copy data from the old table
cursor.execute('''
    INSERT INTO NewCustomers (ID, Name, Email, RegistrationDate)
    SELECT ID, Name, Email, RegistrationDate FROM Customers
''')

# Step 3: Drop the old table
cursor.execute("DROP TABLE IF EXISTS Customers")

# Step 4: Rename the new table
cursor.execute("ALTER TABLE NewCustomers RENAME TO Customers")
```

---

## 5. DATA Operations

### 5.1 Inserting Data (Using Prepared Statements)
```python
connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()

cursor.execute("INSERT INTO Customers (Name, Email) VALUES (?, ?)",
               ("Alice", "alice@example.com"))

connection.commit()
connection.close()
```

### 5.2 Querying Data
```python
connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()

cursor.execute("SELECT * FROM Customers")
customers = cursor.fetchall()
for customer in customers:
    print(customer)

connection.close()
```

### 5.3 Updating Data
```python
connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()

cursor.execute("UPDATE Customers SET Email = ? WHERE ID = ?", 
               ("alice.new@example.com", 1))

connection.commit()
connection.close()
```

### 5.4 Deleting Data
```python
connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()

cursor.execute("DELETE FROM Customers WHERE ID = ?", (1,))

connection.commit()
connection.close()
```

---

## 6. Using Transactions

### 6.1 Enabling Foreign Keys
```python
connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()
cursor.execute("PRAGMA foreign_keys = ON;")
```

### 6.2 Using Transactions for Multiple Queries
```python
try:
    connection = sqlite3.connect('my_database.db')
    cursor = connection.cursor()
    
    cursor.execute("BEGIN TRANSACTION;")
    cursor.execute("INSERT INTO Customers (Name, Email) VALUES (?, ?)",
                   ("John Doe", "john@example.com"))
    cursor.execute("INSERT INTO Customers (Name, Email) VALUES (?, ?)",
                   ("Jane Doe", "jane@example.com"))
    
    connection.commit()
except sqlite3.Error as e:
    print("Error occurred:", e)
    connection.rollback()
finally:
    connection.close()
```

---

## 7. Working with JSON in SQLite
SQLite provides built-in JSON support. You can use the `json_extract()` function to work with JSON data:
```python
cursor.execute("SELECT json_extract('{\"name\": \"Alice\"}', '$.name');")
```

---

## 8. Exporting and Importing Data

### 8.1 Exporting to CSV
```python
import csv

connection = sqlite3.connect('my_database.db')
cursor = connection.cursor()

cursor.execute("SELECT * FROM Customers")
rows = cursor.fetchall()

with open('customers.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow([desc[0] for desc in cursor.description])  # Column names
    writer.writerows(rows)

connection.close()
```

### 8.2 Importing from CSV
```python
with open('customers.csv', 'r') as file:
    reader = csv.reader(file)
    next(reader)  # Skip header row
    
    connection = sqlite3.connect('my_database.db')
    cursor = connection.cursor()
    
    for row in reader:
        cursor.execute("INSERT INTO Customers (ID, Name, Email, RegistrationDate) VALUES (?, ?, ?, ?)", row)
    
    connection.commit()
    connection.close()
```

---

## 9. Best Practices for Using SQLite with Python
- **Use context managers (`with sqlite3.connect() as conn`)** to handle connections safely.
- **Enable foreign key constraints (`PRAGMA foreign_keys = ON`).**
- **Use `?` placeholders to prevent SQL injection.**
- **Use transactions for batch operations.**
- **Index frequently queried columns for better performance.**

---

## 10. Conclusion
This guide introduces the fundamentals of using **SQLite with Python** while following best practices. Continue exploring advanced topics like **triggers, full-text search, and SQLite extensions** to deepen your knowledge! 🚀
