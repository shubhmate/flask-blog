# Shubham Blog

A full-stack blog application built with Flask, SQLAlchemy, and Bootstrap 5. Supports user authentication, admin post management, comments, and contact form via Mailtrap.

## Features

- User registration and login with hashed passwords
- Role-based admin access (create, edit, delete posts)
- Rich text post editor via CKEditor
- Comment system for authenticated users
- Contact form with Mailtrap SMTP
- Rate limiting on login and register routes
- PostgreSQL support for production (SQLite for local dev)

## Tech Stack

- **Backend:** Flask, SQLAlchemy, Flask-Login, Flask-WTF, Flask-Limiter
- **Frontend:** Bootstrap 5, CKEditor
- **Database:** SQLite (dev) / PostgreSQL (production)
- **Deployment:** Render (gunicorn)

## Local Setup

1. Clone the repo:
```
git clone <your_repo_url>
cd shubham-blog
```

2. Create and activate a virtual environment:
```
python -m venv .venv
.venv\Scripts\activate      # Windows
source .venv/bin/activate   # macOS/Linux
```

3. Install dependencies:
```
pip install -r requirements.txt
```

4. Create your `.env` file from the example:
```
copy .env.example .env      # Windows
cp .env.example .env        # macOS/Linux
```

5. Fill in your `.env` values:
```
FLASK_KEY=<generate: python -c "import secrets; print(secrets.token_hex(32))">
FLASK_DEBUG=true
DB_URI=sqlite:///posts.db
MAILTRAP_USER=<your_mailtrap_username>
MAILTRAP_PASS=<your_mailtrap_password>
MAILTRAP_TO=<recipient_email>
```

6. Run the app:
```
python main.py
```

7. Open `http://localhost:5001` in your browser.

## Setting Up Admin

After registering your first user, open the database and run:
```sql
UPDATE users SET role='admin' WHERE id=1;
```
Use DB Browser for SQLite locally, or the Render shell in production.

## Deployment on Render

1. Push the project to GitHub
2. Create a new **Web Service** on [Render](https://render.com) and connect your repo
3. Set the following:
   - **Build command:** `pip install -r requirements.txt`
   - **Start command:** `gunicorn main:app`
4. Create a **PostgreSQL** database on Render and copy the Internal Database URL
5. Add environment variables in Render dashboard (see `.env.example`)

## Environment Variables

| Variable | Description |
|---|---|
| `FLASK_KEY` | Flask secret key for sessions |
| `FLASK_DEBUG` | Set to `false` in production |
| `DB_URI` | Database connection string |
| `MAILTRAP_USER` | Mailtrap SMTP username |
| `MAILTRAP_PASS` | Mailtrap SMTP password |
| `MAILTRAP_TO` | Email address to receive contact messages |
