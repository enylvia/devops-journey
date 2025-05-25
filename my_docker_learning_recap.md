
# 🐳 My DevOps Learning Recap – Docker & Docker Compose

**Date:** 2025-05-25  
**By:** Me (learning to be DevOps-ready 🚀)

---

## 🧱 What I Learned Today

Today was a big day in my DevOps journey. I focused on understanding Docker deeply, and I actually built and ran a Go application with PostgreSQL using Docker Compose. Here’s everything I covered and figured out along the way:

---

### ✅ 1. Dockerfile

I learned how to:
- Write a clean and efficient multi-stage `Dockerfile` using `golang` and `alpine`
- Use `CGO_ENABLED=0`, `GOOS=linux`, and `GOARCH=amd64` to make my Go binary work in Alpine
- Fix common runtime errors like:
  - `no such file or directory`
  - missing `logs/` folder
- Create folders like `/logs` inside the container during build
- Make sure my binary is actually copied and executable

---

### ✅ 2. Docker Compose

Using `docker-compose.yaml`, I managed to:
- Set up two services: `api` (my Go app) and `db` (PostgreSQL)
- Set up environment variables with `.env` and load them via `env_file`
- Expose ports (and avoid conflicts by changing the host port if needed)
- Use named volumes to persist PostgreSQL data
- Automatically load my schema using `/docker-entrypoint-initdb.d/schema.sql`

---

### ✅ 3. Handling .env and Secrets

I learned:
- Not to commit `.env` to Git
- How to remove it with `git rm --cached .env`
- To safely create a `.env.example` for others
- How to load `.env` only in development mode in my Go app:
  ```go
  if os.Getenv("APP_ENV") == "development" {
      godotenv.Load()
  }
  ```

---

### ✅ 4. Debugging & Logs

During testing, I ran into:
- Port conflicts (fixed with `netstat` and changing exposed ports)
- Log file errors because `/logs` didn't exist in the container

I solved them by:
- Creating folders in the Dockerfile
- Using `os.MkdirAll()` in Go to ensure log paths exist at runtime

---

### ✅ 5. Swaggo Integration

I made sure that:
- `swag init` is run before building
- The generated `docs` folder is included in the build context

That way, my Swagger UI and API documentation work inside the Docker container too.

---

### ✅ 6. Running & Stopping Containers

I learned how to:
- Run everything with `docker-compose up --build -d`
- Check container status with `docker ps`
- View logs with `docker logs` and `docker-compose logs`
- Stop and clean up with `docker-compose down` or `docker-compose down -v` if I want to reset everything

---

## 🔚 Final Thoughts

I'm proud of how much I understood today.  
From writing a Dockerfile to managing volumes, handling environment configurations, and debugging runtime errors—this was a full day of real-world Docker usage.

Tomorrow, maybe I'll dive into CI/CD or Docker Swarm or start learning Kubernetes. We'll see. But today? I feel like I leveled up. 💪

---

