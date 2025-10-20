# python-app.pyx
from flask import Flask, render_template_string, request, redirect
import sqlite3
import datetime

app = Flask(__name__)
DB = "expenses.db"

def init_db():
    conn = sqlite3.connect(DB)
    c = conn.cursor()
    c.execute("""
    CREATE TABLE IF NOT EXISTS expenses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        amount REAL NOT NULL,
        category TEXT,
        created_at TEXT
    )
    """)
    conn.commit()
    conn.close()

@app.route("/")
def index():
    conn = sqlite3.connect(DB)
    c = conn.cursor()
    c.execute("SELECT * FROM expenses ORDER BY id DESC")
    data = c.fetchall()
    total = sum([d[2] for d in data]) if data else 0
    conn.close()
    return render_template_string("""
        <h1>💰 Expense Tracker</h1>
        <a href="/add">➕ Add Expense</a><hr>
        <h3>Total Spent: {{ total }} USD</h3>
        {% for e in data %}
            <div style="border:1px solid #ccc; padding:10px; margin:5px;">
                <b>{{ e[1] }}</b> - ${{ e[2] }} ({{ e[3] }})<br>
                <small>📅 {{ e[4] }}</small><br>
                <a href="/delete/{{ e[0] }}">❌ Delete</a>
            </div>
        {% else %}
            <p>No expenses yet.</p>
        {% endfor %}
    """, data=data, total=total)

@app.route("/add"
