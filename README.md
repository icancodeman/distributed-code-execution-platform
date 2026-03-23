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
