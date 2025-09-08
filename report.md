# 🛡️ SQL Injection Playground

This project is a learning playground for understanding SQL Injection. It shows how a login page can be hacked if it is vulnerable, and how to fix it using secure coding practices. The app is built using Flask (Python web framework) and SQLite (lightweight database).

---

## 📂 Project Overview

| Feature             | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Vulnerable Login    | Located at `/`. Uses string formatting to build SQL queries (unsafe).       |
| Secure Login        | Located at `/secure`. Uses parameterized queries to prevent SQL injection.  |
| Database            | SQLite database `users.db` with a `users` table and sample accounts.        |
| Sample Users        | `admin / admin123` and `test / test123`                                     |

---

## ⚡ Setup Instructions

1. Clone the repo:
   git clone https://github.com/your-username/sql-injection-playground.git
   cd sql-injection-playground

2. Create and activate a virtual environment:
   python -m venv venv
   venv\Scripts\activate   # On Windows
   source venv/bin/activate # On Mac/Linux

3. Install Flask:
   pip install flask

4. Run the application:
   python app.py

Now open your browser:
- Vulnerable login → http://127.0.0.1:5000/
- Secure login → http://127.0.0.1:5000/secure

---

## 👤 Default Users in Database

| Username | Password  | Purpose                        |
|----------|-----------|--------------------------------|
| admin    | admin123  | For testing admin login        |
| test     | test123   | For testing normal user login  |

---

## ❌ Vulnerable Login (How SQL Injection Works)

The vulnerable login builds queries like this:

query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
cur.execute(query)

Here the input is directly placed into the query. This is unsafe because attackers can change the SQL logic.

Example attack:
If an attacker enters:
- Username: admin' --
- Password: anything

The query becomes:

SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything'

The two dashes (--) make the rest of the query a comment. This removes the password check and lets the attacker log in as admin without knowing the password. This is a classic SQL Injection attack.

---

## ✅ Secure Login (How to Fix It)

The secure login uses parameterized queries:

query = "SELECT * FROM users WHERE username=? AND password=?"
cur.execute(query, (username, password))

Here the question marks are placeholders. User input is passed separately, not mixed into the SQL. The database treats input as plain data, not as code. Even if someone tries admin' --, it will not break the query. This is the correct way to handle database queries safely.

---

## 🔬 Testing the Difference

| Page               | Behavior                                                                 |
|--------------------|--------------------------------------------------------------------------|
| / (Vulnerable)     | Allows SQL injection. Example: admin' -- bypasses password check.        |
| /secure (Secure)   | Blocks SQL injection. Same input will fail because the query is safe.    |

---

## 🚀 Why This Matters

SQL Injection is one of the most common web vulnerabilities. It can allow attackers to:
- Bypass logins  
- Read sensitive data  
- Delete or modify database contents  
- Take control of entire applications  

The safe coding practice is always:
- Use parameterized queries (placeholders)  
- Never build SQL by concatenating strings  

---

## ⚠️ Disclaimer

This project is for educational purposes only. The vulnerable code is intentionally insecure and must never be used in real applications. The secure version demonstrates the correct way to protect against SQL Injection.

---

## 📖 Summary

| Vulnerable Login             | Secure Login                        |
|------------------------------|-------------------------------------|
| Builds SQL with string input | Uses placeholders (?)                |
| Can be hacked with injection | Blocks injection completely         |
| Example: admin' -- works     | Example: admin' -- does not work    |
| Dangerous in real life       | Safe coding practice                |
