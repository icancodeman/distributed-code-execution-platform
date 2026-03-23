##if code are not visibles copy all codes

1 app.py

from flask import Flask, request, jsonify, render_template
from flask_jwt_extended import JWTManager, create_access_token, jwt_required
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from celery_worker import run_code_task
from celery.result import AsyncResult
app = Flask(__name__)

app.config["JWT_SECRET_KEY"] = "super-secret-key"
jwt = JWTManager(app)

limiter = Limiter(get_remote_address, app=app)

@app.route('/')
def home():
    return render_template("index.html")

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get("username")
    token = create_access_token(identity=username)
    return {"access_token": token}

@app.route('/submit', methods=['POST'])
@jwt_required()
@limiter.limit("5 per minute")
def submit_code():
    code = request.json.get("code")
    task = run_code_task.delay(code)
    return {"task_id": task.id}

@app.route('/result/<task_id>')
def get_result(task_id):
    result = AsyncResult(task_id)
    if result.ready():
        return jsonify(result.result)
    return {"status": "processing"}

if __name__ == '__main__':
    app.run(debug=True)

2 exe.py 

import subprocess
import tempfile
import os

def execute_code(code):
    try:
        with tempfile.NamedTemporaryFile(delete=False, suffix=".py") as f:
            f.write(code.encode())
            file_path = f.name

        result = subprocess.run(
            [
                "docker", "run", "--rm",
                "-v", f"{file_path}:/app/code.py",
                "python:3.9",
                "python", "/app/code.py"
            ],
            capture_output=True,
            text=True,
            timeout=5
        )

        os.remove(file_path)

        return {
            "output": result.stdout,
            "error": result.stderr
        }

    except Exception as e:
        return {"error": str(e)}

3.celery_worker.py

from celery import Celery
from executor import execute_code

celery = Celery(
    'tasks',
    broker='redis://localhost:6379/0',
    backend='redis://localhost:6379/0'
)

@celery.task
def run_code_task(code):
    return execute_code(code)

4.temp/index.html

<!DOCTYPE html>
<html>
<head>
    <title>Code Executor</title>
</head>
<body>

<h2>Run Python Code</h2>

<textarea id="code" rows="10" cols="50">
print("Hello World")
</textarea><br><br>

<button onclick="runCode()">Run</button>

<pre id="result"></pre>

<script>
async function runCode() {
    let code = document.getElementById("code").value;

    let tokenRes = await fetch("/login", {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({username: "test"})
    });

    let tokenData = await tokenRes.json();

    let res = await fetch("/submit", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "Authorization": "Bearer " + tokenData.access_token
        },
        body: JSON.stringify({code})
    });

    let data = await res.json();
    checkStatus(data.task_id);
}

async function checkStatus(id) {
    let res = await fetch("/result/" + id);
    let data = await res.json();

    if (data.status === "processing") {
        setTimeout(() => checkStatus(id), 1000);
    } else {
        document.getElementById("result").innerText =
            data.output || data.error;
    }
}
</script>

</body>
</html>


Requirement 

flask
celery
redis
flask-jwt-extended
flask-limiter


# distributed-code-execution-platform
Scalable backend system for secure code execution using Docker, Celery, and Redis
# 🚀 Distributed Code Execution Platform

A production-style backend system that allows users to execute Python code securely using Docker-based sandboxing.

---

## 🔥 Features

- Secure code execution using Docker
- Asynchronous job processing with Celery + Redis
- JWT Authentication
- Rate limiting (anti-abuse)
- Frontend UI for code execution
- Scalable worker architecture

---

## 🏗️ Architecture

Client → Flask API → Redis Queue → Worker → Docker → Result

---

## ⚙️ Tech Stack

- Python (Flask)
- Celery + Redis
- Docker
- HTML + JavaScript

---

## ▶️ Setup Instructions

1. Install dependencies:
pip install -r requirements.txt

2. Start Redis:
redis-server

3. Start Celery worker:
celery -A celery_worker.celery worker --loglevel=info

4. Run Flask app:
python app.py

5. Open browser:
http://127.0.0.1:5000

---

## 💡 How It Works

- User submits code
- API sends job to Celery queue
- Worker executes code inside Docker container
- Result is returned to user

---

## 🚀 Why This Project

This project demonstrates:
- Backend system design
- Secure sandbox execution
- Distributed architecture
- Production-level thinking

---

## 📌 Future Improvements

- Multi-language support
- Kubernetes scaling
- Advanced monitoring
