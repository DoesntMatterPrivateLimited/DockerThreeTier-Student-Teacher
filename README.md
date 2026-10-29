# Student–Teacher Three-Tier Application

A React frontend, Node.js/Express API, and MySQL database run together with Docker Compose.

## Project structure

```text
.
├── assets/
│   └── Infra.gif                 # Architecture illustration
├── backend/
│   ├── Dockerfile                # Backend container image
│   ├── package.json              # Backend dependencies and scripts
│   ├── package-lock.json
│   └── server.js                 # API routes and database setup
├── database/
│   ├── Dockerfile                # Standalone MySQL image (not used by Compose)
│   └── Dockerfile-NBP            # Alternate/legacy MySQL Dockerfile
├── frontend/
│   ├── public/                   # Static public assets
│   ├── src/                      # React app, pages, and components
│   ├── .env.example              # Frontend API URL for local development
│   ├── Dockerfile                # Builds React and serves it with Nginx
│   ├── nginx.conf                # SPA route fallback
│   └── package.json              # Frontend dependencies and scripts
└── docker-compose.yml            # Defines and connects all three services
```

## Architecture and ports

| Service | Purpose | Host address |
| --- | --- | --- |
| `frontend` | React app served by Nginx | [http://localhost:3000](http://localhost:3000) |
| `backend` | Express API | [http://localhost:3500](http://localhost:3500) |
| `db` | MySQL 8 database | `localhost:3306` |

Compose connects the services on the `three-tier-network` bridge network. The backend waits for MySQL's health check before starting. On startup, the backend creates the `student` and `teacher` tables if they do not already exist. MySQL data is stored in the `mysql_data` named volume.

## Requirements

- Docker Desktop (or Docker Engine) with the Docker Compose v2 plugin
- Git, if you are cloning the project

## Run with Docker Compose

Run these commands from the project root in PowerShell:

```powershell
# Check the Compose configuration
docker compose config

# Build the images and start all services in the background
docker compose up --build -d

# Check service status
docker compose ps
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

The initial build downloads base images and npm dependencies, so it may take a few minutes. The frontend is built with `REACT_APP_API_BASE_URL=http://localhost:3500`.

## Useful Docker commands

```powershell
# Follow logs from every service
docker compose logs -f

# Follow logs from one service
docker compose logs -f backend

# Restart the services
docker compose restart

# Rebuild and restart after changing application code or Docker configuration
docker compose up --build -d

# Stop and remove the containers and Compose network
docker compose down
```

To open a MySQL prompt:

```powershell
docker compose exec db mysql -uroot -prootpassword student_teacher_db
```

At the MySQL prompt, useful inspection commands include:

```sql
SHOW TABLES;
SELECT * FROM student;
SELECT * FROM teacher;
EXIT;
```

The current Compose file uses the development credentials `root` / `rootpassword`. Change them before using this setup outside a local development environment.

### Reset the database

`docker compose down` preserves the `mysql_data` volume. To permanently delete the database contents and start with an empty database, run:

```powershell
docker compose down -v
docker compose up --build -d
```

**Warning:** `docker compose down -v` deletes the database volume and its data.

## API endpoints

The API is available at `http://localhost:3500`:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/health` | Check backend health |
| `GET` | `/health/db` | Check the backend's database connection |
| `GET` | `/student` | List students |
| `POST` | `/addstudent` | Add a student |
| `DELETE` | `/student/:id` | Delete a student |
| `GET` | `/teacher` | List teachers |
| `POST` | `/addteacher` | Add a teacher |
| `DELETE` | `/teacher/:id` | Delete a teacher |

For example, check backend health from PowerShell:

```powershell
Invoke-RestMethod http://localhost:3500/health
```

## Run the apps locally (optional)

For local development, start the database with Compose, then run the backend and frontend from separate terminals.

**Terminal 1 — MySQL:**

```powershell
docker compose up -d db
```

**Terminal 2 — backend:**

```powershell
cd backend
npm ci
$env:DB_HOST = "localhost"
$env:DB_PORT = "3306"
$env:DB_USER = "root"
$env:DB_PASSWORD = "rootpassword"
$env:DB_NAME = "student_teacher_db"
npm start
```

**Terminal 3 — frontend:**

```powershell
cd frontend
npm ci
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
npm start
```

The local React development server opens on [http://localhost:3000](http://localhost:3000). The API URL in `frontend/.env` must point to `http://localhost:3500`.

## Frontend build and tests

Run these from the `frontend` directory:

```powershell
# Create an optimized production build
npm run build

# Run the frontend tests once
npm test -- --watchAll=false
```

<img width="1312" height="693" alt="image" src="https://github.com/user-attachments/assets/687c0713-170c-4e20-86f6-6ccd12c3619f" />
<img width="1319" height="657" alt="image" src="https://github.com/user-attachments/assets/f868f399-96eb-41a6-b77c-c66363d3438b" />
<img width="1312" height="687" alt="image" src="https://github.com/user-attachments/assets/6234c996-7f1f-4803-92bc-b3ad1ea3ea04" />
<img width="1359" height="679" alt="image" src="https://github.com/user-attachments/assets/dbb87c5a-2587-4aed-b40b-2d3e59329bea" />
<img width="1364" height="689" alt="image" src="https://github.com/user-attachments/assets/1f565eb1-1d0f-4820-a98f-53be64b18f07" />
<img width="1365" height="720" alt="image" src="https://github.com/user-attachments/assets/4422ddda-657d-4cc9-bcad-3d4f5d15cd87" />


