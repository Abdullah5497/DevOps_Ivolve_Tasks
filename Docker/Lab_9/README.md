# Lab 9: Containerized Node.js and MySQL Stack Using Docker Compose

This lab demonstrates how to containerize a Node.js application with a MySQL database using Docker Compose, configure environment variables, persist database data using volumes, and verify application health and logs.

---

## Prerequisites

- Docker
- Docker Compose
- Git
- DockerHub account

---

## Step 1: Clone the Application Repository

```bash
git clone https://github.com/Ibrahim-Adel15/kubernets-app.git
cd kubernets-app
```

---

## Step 2: Application Requirements

- Application runs on port **3000**
- Requires MySQL database named `ivolve`
- Uses environment variables for database connection:
  - `DB_HOST`
  - `DB_USER`
  - `DB_PASSWORD`

---

## Step 3: Create `docker-compose.yml`

Create a file named `docker-compose.yml` in the project root:

```yaml
version: "3"
services:

  app_service:
    build: .
    container_name: myapp
    ports:
      - "3000:3000"
    environment:
      DB_HOST: mysql_db
      DB_USER: root
      DB_PASSWORD: root
    depends_on:
      - db_service

  db_service:
    image: mysql:8.0
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ivolve
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

> **Note:** The `depends_on` directive ensures the app service waits for the database service to start before launching.

---

## Step 4: Build and Run the Stack

```bash
docker compose up -d --build
```

This command will:
1. Build the Node.js app image from the local `Dockerfile`
2. Pull the `mysql:8.0` image
3. Start both containers in detached mode

---

## Step 5: Verify Application is Running

Check running containers:

```bash
docker ps
```

Access the application in your browser:

```
http://localhost:3000
```

---

## Step 6: Verify Health Endpoints

```bash
curl http://localhost:3000/health
```

```bash
curl http://localhost:3000/ready
```

Expected responses confirm the app is connected to the database and ready to serve traffic.

---

## Step 7: Verify Application Logs

```bash
docker exec -it myapp cat /app/logs/access.log
```

This prints the access log from inside the running container, showing incoming HTTP requests.

---

## Step 8: Push Image to DockerHub

**Login to DockerHub:**

```bash
docker login
```

**Tag the image:**

```bash
docker tag myapp fatmaahassan/kubernets-app:lab9
```

**Push the image:**

```bash
docker push fatmaahassan/kubernets-app:lab9
```

---

## 📸 Screenshots (Lab 9 Execution Result)

> Add your screenshots here showing:
> - `docker ps` output with both containers running
> - Browser/curl output for `http://localhost:3000`
> - `/health` and `/ready` endpoint responses
> - Access log output from `/app/logs/access.log`
> - DockerHub push confirmation

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                   Docker Network                     │
│                                                      │
│  ┌──────────────────┐      ┌──────────────────────┐  │
│  │   app_service    │      │     db_service        │  │
│  │   (Node.js)      │─────▶│     (MySQL 8.0)       │  │
│  │   Port: 3000     │      │     Port: 3306        │  │
│  └──────────────────┘      └──────────────────────┘  │
│                                        │              │
│                             ┌──────────▼───────────┐  │
│                             │    db_data volume     │  │
│                             │  /var/lib/mysql       │  │
│                             └──────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## Environment Variables Reference

| Variable              | Service     | Value       | Description                     |
|-----------------------|-------------|-------------|---------------------------------|
| `DB_HOST`             | app_service | `mysql_db`  | Hostname of the MySQL container |
| `DB_USER`             | app_service | `root`      | MySQL username                  |
| `DB_PASSWORD`         | app_service | `root`      | MySQL password                  |
| `MYSQL_ROOT_PASSWORD` | db_service  | `root`      | MySQL root password             |
| `MYSQL_DATABASE`      | db_service  | `ivolve`    | Database to create on startup   |

---

## Useful Commands

```bash
# View logs of the app container
docker logs myapp

# View logs of the MySQL container
docker logs mysql_db

# Stop and remove containers
docker compose down

# Stop and remove containers + volumes
docker compose down -v

# Rebuild without cache
docker compose build --no-cache
```
