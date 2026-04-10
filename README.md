# Shubham Blog

A full-stack blog application built with Flask, SQLAlchemy, and Bootstrap 5. Supports user authentication, role-based admin access, comments, contact form via Mailtrap, and rate limiting.

## Features

- User registration and login with hashed passwords (pbkdf2:sha256)
- Role-based admin access (create, edit, delete posts)
- Rich text post editor via CKEditor 4.25.1
- Comment system for authenticated users with XSS sanitization
- Gravatar profile images on comments (SHA256)
- Contact form with Mailtrap SMTP and error handling
- Rate limiting on login and register routes (Flask-Limiter)
- XSS protection via Bleach on all user input
- Token-protected admin setup route
- PostgreSQL support for production (SQLite for local dev)
- Timezone-aware datetime (UTC)
- DB connection pool with pre-ping and timeout
- Dev server binds to `127.0.0.1`, production binds to `0.0.0.0`

## Tech Stack

- **Backend:** Flask, SQLAlchemy, Flask-Login, Flask-WTF, Flask-Limiter, Bleach
- **Frontend:** Bootstrap 5, CKEditor 4.25.1 (CDN)
- **Database:** SQLite (dev) / PostgreSQL (production)
- **Deployment:** Render (gunicorn)

## Security

- Passwords hashed with `pbkdf2:sha256` via Werkzeug
- All user input sanitized with `bleach.clean()`
- Gravatar hashed with SHA256 (not MD5)
- Role-based admin access (`role='admin'` in DB)
- Rate limiting on `/login` and `/register` (10 req/min)
- CSRF protection on all forms via Flask-WTF
- Admin setup route is token-protected and commented out by default
- `.env` excluded from version control via `.gitignore`
- Debug mode controlled via `FLASK_DEBUG` env var

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

## Setting Up Admin (Local)

After registering your first user, open DB Browser for SQLite:
1. Open `instance/posts.db`
2. Go to **Execute SQL** tab
3. Run:
```sql
UPDATE users SET role='admin' WHERE id=1;
```
4. Click **Write Changes**
5. Log out and log back in

## Deployment on Render

1. Push the project to GitHub:
```
git init
git add .
git commit -m "initial commit"
git remote add origin <your_repo_url>
git push -u origin main
```

2. Create a **PostgreSQL** database on Render:
   - Render dashboard → New → PostgreSQL
   - Copy the **External Database URL**

3. Create a new **Web Service** on Render:
   - Connect your GitHub repo
   - **Build command:** `pip install -r requirements.txt`
   - **Start command:** `gunicorn main:app --bind 0.0.0.0:$PORT --timeout 180 --workers 1`
   - **Region:** must match your PostgreSQL region

4. Add environment variables in Render dashboard:
```
FLASK_KEY=<generate with: python -c "import secrets; print(secrets.token_hex(32))">
FLASK_DEBUG=false
DB_URI=<External Database URL from Render PostgreSQL>
MAILTRAP_USER=<your_mailtrap_username>
MAILTRAP_PASS=<your_mailtrap_password>
MAILTRAP_TO=<recipient_email>
```

5. Deploy and wait for `Your service is live 🎉`

## Setting Up Admin (Production)

The `/make-admin/<token>` route is available in the code (commented out). To use it:

1. Uncomment the route in `main.py`
2. Generate a token:
```
python -c "import secrets; print(secrets.token_hex(16))"
```
3. Add `ADMIN_TOKEN=<your_token>` to Render environment variables
4. Push and deploy
5. Register your first user on the live site
6. Visit:
```
https://<your-render-url>/make-admin/<your_token>
```
7. You'll see **"Done - delete ADMIN_TOKEN from environment variables now!"**
8. Delete `ADMIN_TOKEN` from Render env vars and comment the route back out
9. Push again

## Environment Variables

| Variable | Description |
|---|---|
| `FLASK_KEY` | Flask secret key for sessions |
| `FLASK_DEBUG` | Set to `false` in production |
| `PRODUCTION` | Set to `true` on any hosting platform to bind to `0.0.0.0` |
| `DB_URI` | Database connection string (use External URL on Render) |
| `MAILTRAP_USER` | Mailtrap SMTP username |
| `MAILTRAP_PASS` | Mailtrap SMTP password |
| `MAILTRAP_TO` | Email address to receive contact messages |
| `ADMIN_TOKEN` | Token for admin setup route (delete after use) |

## Troubleshooting

### `ModuleNotFoundError: No module named 'flask_limiter'`
```
pip install Flask-Limiter==3.9.0
```

### `ModuleNotFoundError: No module named 'psycopg2'`
Make sure `requirements.txt` has `psycopg2-binary` not `psycopg-binary`:
```
psycopg2-binary==2.9.10
```

### `Install 'email_validator' for email validation support`
```
pip install email-validator==2.2.0
```

### Worker timeout on Render
- Make sure `DB_URI` is set to the **External Database URL** from Render PostgreSQL
- The URL must include the full hostname ending in `.oregon-postgres.render.com:5432`
- Web Service and PostgreSQL must be in the **same region**
- Use `?sslmode=disable` at the end of the internal URL if external URL has DNS issues:
```
postgresql://user:password@dpg-xxxx-a/dbname?sslmode=disable
```

### `could not translate host name` error
- Switch from Internal to External Database URL in `DB_URI` env var on Render

### `postgres://` vs `postgresql://`
SQLAlchemy requires `postgresql://` — the code auto-fixes this, but make sure your URL starts with `postgresql://` in the env var.

### Post body showing raw HTML tags
When adding a post, click the **Source** button (`< >`) in CKEditor toolbar, paste your HTML content directly, then click **Source** again to preview before saving.

### CKEditor security warning
The app loads CKEditor 4.25.1 directly from CDN — no action needed.

### Forbidden (403) on admin routes
Your user's `role` is not set to `admin`. Follow the admin setup steps above.
