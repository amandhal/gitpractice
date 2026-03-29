# 🐳 Day 1 – Introduction & Best Practices

## 📌 Introduction to Docker

Docker is basically a tool that helps us run applications inside something called *containers*. These containers include everything the app needs — like code, libraries, and dependencies — so it works the same everywhere.

Earlier, apps used to break because of “works on my machine” issues, but Docker solves that by keeping environments consistent.

---

## 🤔 Why Docker is Useful

Before Docker, deploying applications was messy:
- Different OS issues  
- Dependency conflicts  
- Manual setup errors  

With Docker:
- Everything is packaged together  
- Runs the same in dev, test, and production  
- Faster deployment  
- Easy to scale  

So basically, Docker makes life easier for both developers and DevOps teams.

---

## ⚙️ Core Concepts 

### 🧱 Docker Image  
An image is like a blueprint or template. It contains everything needed to run an app, but it’s read-only.

### 📦 Docker Container  
A container is the running version of an image. We can start, stop, or delete it anytime.

### 📝 Dockerfile  
This is just a file with instructions to build an image (like what base image to use, what dependencies to install, etc.)

### 💾 Volumes  
Used to store data permanently, even if the container stops.

### 🌐 Networks  
Allow containers to talk to each other.

---

## 🆚 Containers vs Virtual Machines

- **Virtual Machines (VMs)**  
  - Heavy  
  - Each has its own OS  
  - Slower  

- **Containers**  
  - Lightweight  
  - Share the host OS  
  - Faster and efficient  

So containers are more optimized compared to VMs.

---

## 🏗️ Docker Architecture (How It Works)

Docker follows a client-server model:

- **Docker Client** → where we type commands (CLI)  
- **Docker Daemon** → does the actual work (builds, runs containers)  
- **Docker Host** → system where Docker runs  
- **Docker Registry (Docker Hub)** → place to store and download images  

So when we run a command, the client talks to the daemon, and the daemon handles everything.

---

## 📦 Docker Image Registry

A registry is like storage for Docker images.

Types:
- **Public Registry** → anyone can access (e.g., Docker Hub)  
- **Private Registry** → restricted access  

This is useful for sharing images across systems.

---

## 💻 Common Docker Commands 

```bash
docker ps          # list running containers
docker images      # list images
docker build -t app .   # build image
docker run -p 8080:80 app   # run container
docker exec -it <container_id> bash   # access container
```

---

## 🧪 Simple Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

---

## ✅ Best Practices

- Use lightweight images (like alpine)
- Keep images small
- Avoid running containers as root
- Use `.dockerignore` to skip unnecessary files
- Use multi-stage builds when possible

---

## 📌 Final Thoughts

Overall, Docker simplifies how applications are built, shipped, and run.  

**Main idea:**  
Package everything once → run anywhere without issues
