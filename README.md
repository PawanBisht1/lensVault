# LensVault

A photo and video management platform that lets users upload, organize, and access their media from anywhere, 24/7.

## Features

- **Photo & Video Upload** -- Drag-and-drop upload with progress tracking. Supports JPEG, PNG, WebP, HEIC, MP4, MOV, and WebM.
- **Smart Albums** -- Auto-organized by date, location, and people. Create custom albums or let the system do it.
- **Face Recognition** -- Detects and groups photos by the people in them using YOLO object detection.
- **Interactive Map** -- Browse your library on a world map with photo clusters and location-based browsing.
- **Instant Search** -- Full-text search across titles, locations, tags, and people with keyboard shortcut (Cmd+K).
- **Favorites, Archive & Trash** -- Organize photos with soft-delete, restore, and permanent delete.
- **Shared Albums** -- Generate share links for albums with token-based access.
- **EXIF Extraction** -- Automatically extracts camera model, date taken, and GPS coordinates from uploaded photos.
- **Reverse Geocoding** -- Converts GPS coordinates to human-readable location names.
- **Dark Mode** -- Full light/dark theme support across the entire UI.

## Tech Stack

| Layer     | Technology                                          |
|-----------|-----------------------------------------------------|
| Frontend  | Next.js 15, React 19, Tailwind CSS 4, TypeScript    |
| Backend   | FastAPI, SQLAlchemy (async), Pydantic v2             |
| Database  | SQLite (dev) / PostgreSQL (production)               |
| Auth      | JWT (access + refresh tokens), bcrypt password hashing |
| Storage   | Local filesystem or S3/MinIO                         |
| Cache     | Redis (FakeRedis for local dev)                      |
| Images    | Pillow for thumbnails and EXIF                       |
| Geocoding | Nominatim (OpenStreetMap)                            |

## Project Structure

```
LensVault/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app entry point
│   │   ├── config.py            # Settings (env-based)
│   │   ├── database.py          # SQLAlchemy async setup
│   │   ├── models/              # Database models (User, Photo, Album, Person, Face, Tag)
│   │   ├── routers/             # API routes (auth, photos, upload, albums, people, search, map, trash, settings)
│   │   ├── schemas/             # Pydantic request/response schemas
│   │   ├── services/            # Business logic layer
│   │   ├── middleware/          # Auth middleware, rate limiting
│   │   └── utils/               # EXIF extraction, geocoding, pagination
│   ├── alembic/                 # Database migrations
│   ├── tests/                   # Backend tests
│   ├── uploads/                 # Local file storage (auto-created)
│   ├── requirements.txt
│   ├── Dockerfile
│   └── docker-compose.yml
├── frontend/
│   ├── src/
│   │   ├── app/                 # Next.js pages (login, register, albums, upload, etc.)
│   │   ├── components/          # Reusable UI components
│   │   ├── lib/api.ts           # API client with auto token refresh
│   │   └── hooks/               # Custom React hooks
│   ├── public/                  # Static assets (logo)
│   └── package.json
└── README.md
```

---

## How to Run the Project

You need to run **two separate terminals** -- one for the backend (API server) and one for the frontend (UI).

---

### Step 1: Clone / Open the Project

Open your terminal and navigate to the project folder:

```bash
cd path/to/LensVault
```

---

### Step 2: Start the Backend (Terminal 1)

Open **Terminal 1** and run these commands one by one:

```bash
# Go to the backend folder
cd backend

# Create a Python virtual environment (only first time)
python -m venv venv
```

Activate the virtual environment:

```bash
# On Windows:
venv\Scripts\activate

# On Mac/Linux:
source venv/bin/activate
```

You should see `(venv)` at the start of your terminal line. Now install dependencies and start the server:

```bash
# Install all Python packages (only first time, or after pulling new changes)
pip install -r requirements.txt

# Start the backend server
uvicorn app.main:app --reload --port 8000
```

You should see:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
INFO:     Application startup complete.
```

To verify, open http://localhost:8000/health in your browser. You should see:
```json
{"status": "ok", "storage": "local", "database": "sqlite"}
```

> The database (`lensvault.db`) and `uploads/` folder are auto-created on first startup.

---

### Step 3: Start the Frontend (Terminal 2)

Open a **new terminal** (keep the backend terminal running!) and run:

```bash
# Go to the frontend folder
cd frontend

# Install npm packages (only first time, or after pulling new changes)
npm install

# Start the frontend dev server
npm run dev
```

You should see:
```
  ▲ Next.js 15.x
  - Local:   http://localhost:3000
```

---

### Step 4: Open the App

Open http://localhost:3000 in your browser.

- You will see the **Landing Page**
- Click **"Get Started"** or go to http://localhost:3000/register to create an account
- After registering, you will be redirected to the **Photos** page
- Start uploading your photos and videos!

---

### Quick Summary

| What           | Command                                    | URL                    |
|----------------|--------------------------------------------|------------------------|
| Backend        | `uvicorn app.main:app --reload --port 8000` | http://localhost:8000  |
| Frontend       | `npm run dev`                               | http://localhost:3000  |
| Health Check   | --                                          | http://localhost:8000/health |
| API Docs       | --                                          | http://localhost:8000/docs   |

---

## Environment Variables

Copy `backend/.env.example` to `backend/.env` and adjust as needed:

| Variable                      | Default                  | Description                        |
|-------------------------------|--------------------------|------------------------------------|
| `APP_ENV`                     | `development`            | App environment                    |
| `SECRET_KEY`                  | (change in production)   | JWT signing key                    |
| `DATABASE_URL`                | `sqlite+aiosqlite:///./lensvault.db` | Database connection string |
| `REDIS_URL`                   | `fake`                   | Redis URL (`fake` uses FakeRedis)  |
| `STORAGE_BACKEND`             | `local`                  | `local` or `minio` (S3)           |
| `UPLOAD_DIR`                  | `./uploads`              | Local upload directory             |
| `FACE_DETECTION_ENABLED`      | `false`                  | Enable YOLO face detection         |

Frontend uses `NEXT_PUBLIC_API_URL` (defaults to `http://localhost:8000`).

## API Endpoints

| Group    | Endpoint                    | Method | Description                    |
|----------|-----------------------------|--------|--------------------------------|
| Auth     | `/api/auth/register`        | POST   | Register new user              |
| Auth     | `/api/auth/login`           | POST   | Login (returns JWT tokens)     |
| Auth     | `/api/auth/refresh`         | POST   | Refresh access token           |
| Auth     | `/api/auth/logout`          | POST   | Logout (revoke refresh token)  |
| Auth     | `/api/auth/me`              | GET    | Get current user profile       |
| Photos   | `/api/photos`               | GET    | List photos (with filters)     |
| Photos   | `/api/photos/by-month`      | GET    | Photos grouped by month        |
| Photos   | `/api/photos/{id}`          | GET    | Get single photo               |
| Photos   | `/api/photos/{id}`          | PATCH  | Update photo                   |
| Photos   | `/api/photos/{id}`          | DELETE | Trash a photo                  |
| Photos   | `/api/photos/{id}/restore`  | POST   | Restore from trash             |
| Photos   | `/api/photos/bulk`          | POST   | Bulk actions on photos         |
| Upload   | `/api/upload`               | POST   | Upload single file             |
| Upload   | `/api/upload/bulk`          | POST   | Upload multiple files          |
| Albums   | `/api/albums`               | GET    | List all albums                |
| Albums   | `/api/albums`               | POST   | Create album                   |
| Albums   | `/api/albums/{id}`          | GET    | Get album with photos          |
| Albums   | `/api/albums/{id}/photos`   | POST   | Add photos to album            |
| Albums   | `/api/albums/{id}/share`    | POST   | Generate share link            |
| People   | `/api/people`               | GET    | List detected people           |
| People   | `/api/people/{id}`          | PATCH  | Update person name             |
| People   | `/api/people/merge`         | POST   | Merge two people               |
| Search   | `/api/search`               | GET    | Search photos                  |
| Map      | `/api/map/clusters`         | GET    | Get map clusters               |
| Map      | `/api/map/photos`           | GET    | Get photos in map bounds       |
| Trash    | `/api/trash`                | GET    | List trashed photos            |
| Trash    | `/api/trash/restore-all`    | POST   | Restore all trashed photos     |
| Trash    | `/api/trash/empty`          | DELETE | Permanently delete all trash   |
| Settings | `/api/settings`             | GET    | Get user settings              |
| Settings | `/api/settings`             | PATCH  | Update user settings           |
| Settings | `/api/settings/storage`     | GET    | Get storage usage info         |
| Health   | `/health`                   | GET    | Server health check            |

Full interactive API docs available at http://localhost:8000/docs (Swagger UI).

## Docker (Production)

```bash
cd backend
docker-compose up --build
```

This starts the FastAPI backend with PostgreSQL and Redis.
