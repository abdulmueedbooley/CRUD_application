# How to make a basic Flask Application

Task Master — Modern Flask CRUD Application
Comprehensive Developer Guide: Modern Commands, File Creation, Source Code & Technical Reasoning
This developer guide documents the complete step-by-step process of building a full-stack Flask CRUD application from scratch. It reflects modern Python 3.14+ standards, Flask-SQLAlchemy 3.0+ conventions, and Windows PowerShell workflows.
Step 1: Project Setup & PowerShell Environment
Create the project directory, configure PowerShell execution policies, set up an isolated virtual environment, and install dependencies.
📄 PowerShell Setup Commands
# 1. Create project folder and navigate inside
mkdir CRUD_application
cd CRUD_application

# 2. Create virtual environment
python -m venv env

# 3. Allow script execution in PowerShell (CurrentUser scope)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 4. Activate virtual environment
.\env\Scripts\Activate.ps1

# 5. Install required packages
pip install flask flask-sqlalchemy

💡 Technical Reasoning & Key Concepts
• python -m venv env: Creates an isolated virtual environment ('env') so dependencies stay inside this project without polluting system-wide Python installation.
• Set-ExecutionPolicy: Windows blocks unverified PowerShell scripts by default (throwing an UnauthorizedAccess error). Setting RemoteSigned for CurrentUser safely permits local activation scripts (Activate.ps1) to run without administrative privilege escalation.
• (env) Prompt Indicator: Confirms that terminal commands now execute using the environment's isolated Python binary rather than global Python.
Step 2: Main Application File & Database Model (crud.py)
Create crud.py in the root folder to initialize Flask, define database configuration settings, and establish the data schema.
📄 crud.py (Initial Setup & Model)
from flask import Flask, render_template, request, redirect
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)

# 1. Database Configuration (MUST come BEFORE initializing SQLAlchemy)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///test.db'

# 2. Initialize Extension
db = SQLAlchemy(app)

# 3. Database Model Definition
class Todo(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(200), nullable=False)
    date_created = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<Task {self.id}>'

💡 Technical Reasoning & Architecture
• Configuration Order: app.config['SQLALCHEMY_DATABASE_URI'] must be declared BEFORE initializing db = SQLAlchemy(app). Initializing db first causes SQLAlchemy to read an empty URI configuration, resulting in engine instantiation failures.
• sqlite:///test.db: Uses a relative path (3 forward slashes) so SQLite stores data locally inside the project.
• nullable=False: Enforces database validation so users cannot post blank tasks into the database.
• __repr__ Method: Returns a clean string representation (e.g., <Task 1>) whenever a model instance is queried, making debugging straightforward.
Step 3: Modern Database Initialization (Flask REPL)
Initialize the database tables via the Python interactive shell within an explicit application context.
📄 Interactive Terminal Commands
# 1. Launch interactive Python shell inside activated environment
python

# 2. Execute initialization inside application context (Python REPL >>>)
>>> from crud import app, db
>>> with app.app_context():
...     db.create_all()
... 
>>> exit()

💡 Technical Reasoning & Context
• Interactive Shell vs. Terminal: from crud import db is Python code, not a PowerShell command. Executing it directly in PowerShell fails with '[Errno 2] No such file or directory'. It must run inside the interactive Python REPL (>>>).
• app.app_context(): Flask-SQLAlchemy 3.0+ requires an active application context to interact with the database engine. Running db.create_all() without 'with app.app_context():' throws a RuntimeError: Working outside of application context.
• instance/test.db: Modern Flask automatically routes SQLite files into an 'instance/' directory to separate runtime data from core project code and prevent accidental git commits.
Step 4: HTML Templating & Custom Styling
Create a modular layout using Jinja2 template inheritance and custom CSS grid lines.
📄 templates/base.html (Master Layout)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
    {% block head %}{% endblock %}
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>

📄 static/css/main.css (Styles)
body {
    font-family: sans-serif;
    margin: 20px;
}

/* Explicit Table Grid Lines */
table, th, td {
    border: 1px solid black;
    border-collapse: collapse;
}

th, td {
    padding: 8px 12px;
    text-align: center;
}

💡 Technical Reasoning & Frontend Setup
• url_for('static', filename='...'): Dynamically resolves asset paths, ensuring CSS/JS references remain intact regardless of nested routing structures.
• border-collapse: collapse;: Standard HTML tables have no visible borders by default. Merging borders creates clean, professional single-line table grids.
Step 5: Route Handlers & Full CRUD Implementation
Add complete Create, Read, Update, and Delete handlers to crud.py, along with corresponding index and update template views.
📄 crud.py (Complete Full-Stack Source Code)
from flask import Flask, render_template, request, redirect
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///test.db'
db = SQLAlchemy(app)

class Todo(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(200), nullable=False)
    date_created = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<Task {self.id}>'

# --- READ & CREATE ROUTE ---
@app.route('/', methods=['POST', 'GET'])
def index():
    if request.method == 'POST':
        task_content = request.form['content']
        new_task = Todo(content=task_content)

        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/')
        except:
            return 'There was an issue adding your task'
    else:
        tasks = Todo.query.order_by(Todo.date_created).all()
        return render_template('index.html', tasks=tasks)

# --- DELETE ROUTE ---
@app.route('/delete/<int:id>')
def delete(id):
    task_to_delete = Todo.query.get_or_404(id)

    try:
        db.session.delete(task_to_delete)
        db.session.commit()
        return redirect('/')
    except:
        return 'There was a problem deleting that task'

# --- UPDATE ROUTE ---
@app.route('/update/<int:id>', methods=['GET', 'POST'])
def update(id):
    task = Todo.query.get_or_404(id)

    if request.method == 'POST':
        task.content = request.form['content']

        try:
            db.session.commit()
            return redirect('/')
        except:
            return 'There was an issue updating your task'
    else:
        return render_template('update.html', task=task)

if __name__ == "__main__":
    app.run(debug=True)

📄 templates/index.html (Main View)
{% extends 'base.html' %}

{% block head %}
<title>Task Master</title>
{% endblock %}

{% block body %}
<div class="content">
    <h1>Task Master</h1>

    {% if tasks|length < 1 %}
        <h4>There are no tasks. Create one below!</h4>
    {% else %}
        <table>
            <tr>
                <th>Task</th>
                <th>Added</th>
                <th>Actions</th>
            </tr>
            {% for task in tasks %}
                <tr>
                    <td>{{ task.content }}</td>
                    <td>{{ task.date_created.date() }}</td>
                    <td>
                        <a href="/delete/{{ task.id }}">Delete</a>
                        <br>
                        <a href="/update/{{ task.id }}">Update</a>
                    </td>
                </tr>
            {% endfor %}
        </table>
    {% endif %}

    <form action="/" method="POST">
        <input type="text" name="content" id="content" required>
        <input type="submit" value="Add Task">
    </form>
</div>
{% endblock %}

📄 templates/update.html (Edit View)
{% extends 'base.html' %}

{% block head %}
<title>Update Task</title>
{% endblock %}

{% block body %}
<div class="content">
    <h1>Update Task</h1>
    <form action="/update/{{ task.id }}" method="POST">
        <input type="text" name="content" id="content" value="{{ task.content }}">
        <input type="submit" value="Update">
    </form>
</div>
{% endblock %}

💡 Technical Reasoning & Handler Design
• try / except Blocks: Python syntax requires every try: block to be paired with an except: or finally: block. Omission triggers a syntax/parsing error ('Try statement must have at least one except or finally clause').
• get_or_404(id): Safely queries records by primary key. If a user enters an invalid ID in the URL, Flask automatically throws an HTTP 404 response rather than crashing with an unhandled 500 server error.
• db.session.commit(): Changes (adding, modifying, deleting) remain in temporary memory until explicitly committed to disk.
Step 6: Git Setup & GitHub Repository Deployment
Prepare project dependencies, configure ignored file rules, and push source code to GitHub.
📄 .gitignore Configuration
env/
.venv/
__pycache__/
instance/
*.db

📄 Git Terminal Commands
# 1. Freeze active dependencies
pip freeze > requirements.txt

# 2. Initialize Git and commit code
git init
git add .
git commit -m "Initial commit of complete Flask CRUD application"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git push -u origin main

💡 Technical Reasoning & Deployment
• Static vs. Dynamic Hosting: GitHub Pages hosts static web content (HTML/CSS/JS) and cannot run Python application servers or manage SQL databases. GitHub Repositories store source code, while platforms like Render or PythonAnywhere host live Python instances.
• .gitignore Filtering: Keeps large virtual environments (env/), temporary Python bytecode (__pycache__), and local SQLite binary databases (instance/) off public repositories.




