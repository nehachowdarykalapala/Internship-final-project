"""
# 🛡️ SQL Injection Playground

This project is a **learning playground** to understand SQL Injection. It demonstrates:

1. ❌ Vulnerable login at `/` (unsafe string formatting)
2. ✅ Secure login at `/secure` (parameterized queries)
3. Database setup with sample users: `admin/admin123`, `test/test123`

⚠️ DISCLAIMER: This code is for educational purposes only. Never use the vulnerable login in production.

---

## 📂 Project Overview

- Vulnerable Login: `/` → Unsafe SQL, can be hacked
- Secure Login: `/secure` → Safe SQL using placeholders
- Database: SQLite (`users.db`) with `users` table
- Default users: admin/admin123, test/test123
"""

from flask import Flask, request, render_template_string, g
import sqlite3
import os

app = Flask(__name__)
DATABASE = 'users.db'

# -------------------------------
# Database Setup
# -------------------------------
def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    return db

def init_db():
    if not os.path.exists(DATABASE):
        conn = sqlite3.connect(DATABASE)
        cur = conn.cursor()
        cur.execute('''
            CREATE TABLE users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE NOT NULL,
                password TEXT NOT NULL
            )
        ''')
        # Sample users
        cur.execute("INSERT INTO users (username,password) VALUES ('admin','admin123')")
        cur.execute("INSERT INTO users (username,password) VALUES ('test','test123')")
        conn.commit()
        conn.close()

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

# -------------------------------
# HTML Template for Login
# -------------------------------
login_template = """
<h2>{{title}}</h2>
<form method="POST">
  Username: <input type="text" name="username"><br><br>
  Password: <input type="password" name="password"><br><br>
  <input type="submit" value="Login">
</form>
<p>{{message}}</p>
"""

# -------------------------------
# ❌ Vulnerable Login
# -------------------------------
@app.route('/', methods=['GET', 'POST'])
def vulnerable_login():
    message = ''
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        # Vulnerable query: directly inserts user input
        query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
        cur = get_db().cursor()
        try:
            cur.execute(query)
            result = cur.fetchone()
            if result:
                message = f"Welcome {username}! (Vulnerable login)"
            else:
                message = "Login Failed"
        except Exception as e:
            message = f"Error: {e}"

    return render_template_string(login_template, title="Vulnerable Login", message=message)

"""
SQL Injection Example (Vulnerable Login):
Input:
    Username: admin' --
    Password: anything
Query becomes:
    SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything'
The '--' comments out password check → attacker can log in without knowing password.
"""

# -------------------------------
# ✅ Secure Login
# -------------------------------
@app.route('/secure', methods=['GET', 'POST'])
def secure_login():
    message = ''
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        # Secure query: parameterized
        query = "SELECT * FROM users WHERE username=? AND password=?"
        cur = get_db().cursor()
        cur.execute(query, (username, password))
        result = cur.fetchone()
        if result:
            message = f"Welcome {username}! (Secure login)"
        else:
            message = "Login Failed"

    return render_template_string(login_template, title="Secure Login", message=message)

"""
Even if attacker enters: admin' --
Parameterized query treats input as data, not code → SQL Injection fails.
"""

# -------------------------------
# Instructions / Setup (README style)
# -------------------------------
"""
Setup Instructions:
1. Clone repo:
   git clone https://github.com/your-username/sql-injection-playground.git
   cd sql-injection-playground

2. Create virtual environment:
   python -m venv venv
   venv\Scripts\activate   # Windows
   source venv/bin/activate # Mac/Linux

3. Install Flask:
   pip install flask

4. Run the app:
   python app.py

Open in browser:
- Vulnerable login → http://127.0.0.1:5000/
- Secure login → http://127.0.0.1:5000/secure

Default users:
- admin / admin123
- test / test123

Testing SQL Injection:
- Vulnerable login: admin' -- bypasses password
- Secure login: admin' -- fails
"""

# -------------------------------
# Run Application
# -------------------------------
if __name__ == '__main__':
    init_db()
    app.run(debug=True)
