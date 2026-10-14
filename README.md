# SAFEREPORT — AI-Assisted Crime Reporting & Community Safety Platform

> **Report Safely. Track Transparently. Build Safer Communities.**

---

## ⚠️ Important Disclaimers

- SAFEREPORT is **NOT** an official government or police portal.
- Submissions here do **NOT** constitute a legal FIR (First Information Report).
- This platform does **NOT** automatically dispatch emergency services.
- AI suggestions are **model outputs only** — a human reviewer always makes the final decision.
- **For emergencies, call 112 (India) immediately.**

---

## Features

| Feature | Status |
|---|---|
| Multi-step anonymous/authenticated incident reporting | ✅ |
| Unique public reference + cryptographic tracking token | ✅ |
| Secure report status tracking | ✅ |
| AI incident classification (TF-IDF + Logistic Regression) | ✅ |
| Duplicate candidate detection (cosine similarity) | ✅ |
| Reviewer dashboard with status management | ✅ |
| Internal reviewer notes (hidden from citizens) | ✅ |
| Admin dashboard — user roles, audit logs, stats | ✅ |
| Citizen dashboard with notifications | ✅ |
| Private evidence uploads with SHA-256 integrity | ✅ |
| Full audit log | ✅ |
| English + Tamil language switcher (UI strings) | 🔜 i18n files ready |
| Safety map (Leaflet/OSM) | 🔜 Optional, privacy-gated |
| PostgreSQL (production) / SQLite (development) | ✅ |
| JWT authentication, RBAC, security headers | ✅ |

---

## Technology Stack

### Frontend
- React 18 + TypeScript
- Vite 8
- Tailwind CSS v4
- React Router v6
- Axios
- Lucide React icons

### Backend
- Python 3.14 + FastAPI
- SQLAlchemy 2 + Alembic
- Pydantic v2
- Uvicorn
- passlib[bcrypt] + python-jose (JWT)
- slowapi (rate limiting)

### Database
- **Development**: SQLite (zero setup)
- **Production**: PostgreSQL 16

### AI / ML
- scikit-learn (TF-IDF + Logistic Regression)
- Trained on synthetic labelled data (clearly marked)
- Configurable confidence threshold (default: 0.55)

---

## Prerequisites

| Tool | Minimum Version |
|---|---|
| Python | 3.10+ |
| Node.js | 18+ |
| npm | 9+ |
| PostgreSQL | 14+ (production only) |
| Docker + Docker Compose | Optional |

---

## Quick Start (Windows PowerShell — Development with SQLite)

### 1. Clone / open the project
```powershell
cd "d:\Projects\PD LAB\SAFEREPORT"
```

### 2. Backend setup
```powershell
cd backend
python -m venv venv
.\venv\Scripts\pip install -r requirements.txt
Copy-Item .env.example .env
# Edit .env if needed — SQLite is the default, no DB setup required
.\venv\Scripts\uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Backend starts at: **http://localhost:8000**  
API docs: **http://localhost:8000/api/docs**

### 3. Frontend setup (new terminal)
```powershell
cd "d:\Projects\PD LAB\SAFEREPORT\frontend"
npm install
Copy-Item .env.example .env
npm run dev
```

Frontend starts at: **http://localhost:5173**

---

## Environment Variables

### Backend (`.env`)
| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `sqlite:///./safereport.db` | SQLite for dev, PostgreSQL URI for prod |
| `SECRET_KEY` | (change this!) | JWT signing secret — 64+ char random hex |
| `ALGORITHM` | `HS256` | JWT algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `60` | Token lifetime |
| `UPLOAD_DIR` | `./uploads` | Private file storage directory |
| `MAX_UPLOAD_SIZE_MB` | `10` | Max file size per upload |
| `ALLOWED_MEDIA_TYPES` | jpeg,png,gif,webp,mp4,webm | Comma-separated allowed MIME types |
| `ADMIN_EMAIL` | `admin@safereport.local` | Seeded admin email |
| `ADMIN_PASSWORD` | `Admin@1234` | **Change before deployment!** |
| `CORS_ORIGINS` | `http://localhost:5173` | Comma-separated allowed origins |

### Frontend (`.env`)
| Variable | Default | Description |
|---|---|---|
| `VITE_API_BASE_URL` | `` (empty) | API base URL — empty uses Vite proxy |

---

## Default Credentials

| Role | Email | Password |
|---|---|---|
| Admin | `admin@safereport.local` | `Admin@1234` |

To create a reviewer account: register normally, then promote via Admin Dashboard → Users → change role to `reviewer`.

---

## Production: PostgreSQL Setup

```powershell
# Using Docker
docker-compose up -d db

# OR install PostgreSQL locally and:
# CREATE USER safereport_user WITH PASSWORD 'changeme';
# CREATE DATABASE safereport_db OWNER safereport_user;
```

Update `.env`:
```
DATABASE_URL=postgresql://safereport_user:changeme@localhost:5432/safereport_db
SECRET_KEY=<64-character-random-hex-string>
ADMIN_PASSWORD=<strong-production-password>
```

---

## Database Migrations (Alembic)

The app auto-creates tables on startup in development.  
For production, use Alembic migrations:

```powershell
cd backend

# Initialize Alembic (first time only)
.\venv\Scripts\alembic init alembic

# Create a migration
.\venv\Scripts\alembic revision --autogenerate -m "initial schema"

# Apply migrations
.\venv\Scripts\alembic upgrade head

# Check current revision
.\venv\Scripts\alembic current
```

---

## AI Model Setup

The classifier trains automatically on first run using **synthetic data**.

```powershell
# To retrain manually:
.\venv\Scripts\python -c "from app.ai.classifier import train_and_save; train_and_save()"

# Model saved to: ml/models/classifier.pkl
```

**To use real data:** replace the `SYNTHETIC_DATA` list in `backend/app/ai/classifier.py` with properly labeled incident descriptions. Retrain and test before deploying.

**Supported languages:** English only in the baseline. Tamil text will score low confidence (→ `Other/Uncertain`, flagged for manual review).

---

## Running Tests

### Backend tests
```powershell
cd "d:\Projects\PD LAB\SAFEREPORT\backend"
.\venv\Scripts\pytest tests/ -v
```

### Frontend type check
```powershell
cd "d:\Projects\PD LAB\SAFEREPORT\frontend"
npx tsc --noEmit
```

---

## Security Notes

1. **Change `SECRET_KEY`** before any deployment. Use: `python -c "import secrets; print(secrets.token_hex(32))"`
2. **Change `ADMIN_PASSWORD`** before deployment.
3. Tracking tokens are **never stored in plain text** — only SHA-256 hashes are stored.
4. Evidence files are stored with **random filenames** and **never publicly accessible**.
5. All passwords are hashed with **bcrypt**.
6. **CORS** is restricted to configured origins only.
7. Security headers (`X-Content-Type-Options`, `X-Frame-Options`, etc.) are applied to all responses.
8. Rate limiting is applied via **slowapi**.
9. **SQL injection** is prevented through SQLAlchemy ORM parameterized queries.
10. **Path traversal** is prevented — only whitelisted file extensions are stored.

---

## Known Limitations

| Limitation | Notes |
|---|---|
| AI model trained on synthetic data | Replace with real labeled data for production |
| Tamil AI classification unsupported | Low confidence → manual review |
| No email/SMS notifications | In-app only; email requires SMTP config |
| No malware scanning | Documented limitation; file type + size validated |
| Safety map not implemented | Requires privacy-gating before enabling |
| No rate limiting on registration | Add slowapi decorator for production |
| SQLite not recommended for production | Use PostgreSQL |

---

## Project Structure

```
SAFEREPORT/
├── frontend/              React + Vite + Tailwind frontend
│   └── src/
│       ├── pages/         All page components
│       ├── components/    Shared UI components
│       ├── contexts/      Auth context
│       └── services/      API client
├── backend/               FastAPI backend
│   └── app/
│       ├── api/           Route handlers
│       ├── ai/            Classifier + similarity
│       ├── core/          Config
│       ├── db/            DB session
│       ├── models/        SQLAlchemy models
│       ├── schemas/       Pydantic schemas
│       └── security/      Auth + dependencies
├── ml/models/             Trained model artifacts
├── docs/                  Architecture and threat model
├── docker-compose.yml     PostgreSQL + backend
└── README.md
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `bcrypt` error on startup | Run `pip install "bcrypt==4.2.1"` |
| `ImportError: email-validator` | Run `pip install "pydantic[email]"` |
| Frontend shows blank page | Check `npm run dev` is running, check browser console |
| 401 on API calls | JWT expired — log out and log back in |
| Database locked (SQLite) | Only one writer at a time; use PostgreSQL for production |
| CORS errors | Add frontend URL to `CORS_ORIGINS` in `.env` |
