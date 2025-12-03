# **Docker Compose for Multi-Container Apps**

If Docker is the tool for running containers, then **Docker Compose** is the tool that lets you orchestrate them like a conductor leading an orchestra.

When your app has:

- a backend
- a frontend
- a database
- a cache
- maybe a message queue

…running each of them manually with 20 command-line flags is annoying.

Compose fixes that.

---

# **What Is Docker Compose, Really?**

Docker Compose is a tool that reads a single file (`docker-compose.yml`) that describes:

- what containers your app needs
- how they connect
- what volumes they use
- what environment variables they load
- what ports to expose

Imagine onboarding a new developer:

Without Compose:
“Install Node. Install Redis. Install Postgres. Run this, configure that…”

With Compose:
“Just run:

```sh
docker compose up
```

Everything will magically run.”

---

# **Structure of a docker-compose.yml**

This is the foundation:

```yaml
version: "3.9"

services:
  service_name:
    image: some-image
    ports:
      - "host:container"
    environment:
      - KEY=value
    volumes:
      - local_path:container_path
```

Think of **services** as Lego blocks each block is a container, and Compose snaps them together.

---

# **Example 1: Simple Web + Database**

Let’s build the classic pair: Node.js and MySQL.

```yaml
version: "3.9"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - DB_USER=root
      - DB_PASS=secret
    depends_on:
      - db

  db:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=secret
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### What’s happening?

- `app` waits for `db` (thanks to `depends_on`)
- `db` has a persistent volume so your data doesn’t disappear every time you restart
- Both services automatically share the same network
  → `app` can connect to `db` using **hostname: db**

---

# **Networking Without Pain**

In Compose:

- Every service is automatically in the same network
- They _can talk to each other by service name_

Example:

```js
mysql.createConnection({
  host: "db", // NOT localhost
  user: "root",
  password: "secret",
});
```

This is one of the biggest advantages of Compose:
**No manual network configuration. Everything just works.**

---

# **Key Features You Should Know**

## 1. **Environment Variables**

You can inline them:

```yaml
environment:
  - REDIS_HOST=redis
```

Or load from a `.env` file:

```
REDIS_PASSWORD=abcd1234
```

In Compose:

```yaml
env_file:
  - .env
```

Cleaner, safer, easier.

---

## 2. **Volumes: Save Your Data**

Docker containers are temporary.
Volumes make your data permanent.

```yaml
volumes:
  - app_data:/app/logs
```

Or bind mount from your machine:

```yaml
volumes:
  - ./src:/app/src
```

Perfect for local development.

---

## 3. **depends_on: Control Startup Order**

Example:

```yaml
depends_on:
  - redis
  - db
```

Reminder:
`depends_on` ensures startup order, NOT readiness.
If you need readiness, use a wait script or healthchecks.

---

## 4. **Healthchecks: Know If a Service Is Truly Ready**

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 10s
  timeout: 2s
  retries: 5
```

Works great with orchestrations.

---

# **Example 2: Full Stack App (Nginx + Backend + Database)**

A 3-service setup:

- Nginx as reverse proxy
- Node.js API
- Postgres database

```yaml
version: "3.9"

services:
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - api

  api:
    build: ./api
    environment:
      - DB_HOST=postgres
    ports:
      - "3000:3000"
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=1234
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Now, everything works together with just:

```sh
docker compose up --build
```

---

# **Useful Docker Compose Commands**

### Start everything

```sh
docker compose up
```

### Start in detached mode

```sh
docker compose up -d
```

### Stop everything

```sh
docker compose down
```

### See running services

```sh
docker compose ps
```

### Rebuild images

```sh
docker compose up --build
```

### Remove everything (including volumes)

```sh
docker compose down -v
```

### View logs

```sh
docker compose logs -f api
```

---

# **Scaling with Compose**

One backend? Boring.
Let’s run **5 replicas**:

```sh
docker compose up -d --scale api=5
```

Now you have 5 containers for that service.

(Though for real scaling → Kubernetes. Compose is more dev-friendly.)

---

# **Advanced Patterns You Should Know**

### 1. Multi-network setup

Separate frontend and backend networks.

### 2. Multi-compose files

E.g. `docker-compose.override.yml`

### 3. Using profiles

Turn services on/off depending on environment.

```yaml
profiles: ["dev"]
```

### 4. Named networks

Useful for connecting external containers.

### 5. Overriding environment values with `.env`

---

# **Final Thoughts**

Docker Compose is one of those tools that:

- makes development easier
- removes onboarding pain
- organizes your services
- keeps your environment consistent
- helps you think in multi-container architecture

Once you understand Docker Compose, you’ll stop asking:

> “How do I run this app?”

And instead confidently say:

> “Just run `docker compose up`. It’s all there.”

Thank you!
