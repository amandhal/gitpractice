# Contanerize & Deploy 3-Tier MERN Stack App Using Docker

#### Step 1: Create Dockerfile for Backend Tier
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --chown=node:node . .
USER node
EXPOSE 5050
CMD ["node", "server.js"]
```

#### Step 2: Build & Push Backend Image to Docker Hub
```bash
docker build -t amandhal/backend:1.0 .
docker push amandhal/backend:1.0
```
<img width="1521" height="655" alt="image" src="https://github.com/user-attachments/assets/c2642410-6faf-4bac-a5ec-330284baa879" />

#### Step 3: Create Dockerfile for Frontend Tier
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### Step 4: Build & Push Frontend Image to Docker Hub
```bash
docker build -t amandhal/frontend:1.0 .
docker push amandhal/frontend:1.0
```
<img width="1540" height="687" alt="image" src="https://github.com/user-attachments/assets/a091bbc6-e2d2-44a2-b20c-c184686bcdbc" />

#### Step 5: Deploy application using docker compose
This project uses **Docker Compose** to orchestrate a full MERN stack with three services: `frontend`, `backend`, and `mongodb` — all connected via a shared custom network.

---

### 📋 Prerequisites

- [Docker](https://www.docker.com/get-started) installed
- [Docker Compose](https://docs.docker.com/compose/install/) installed

---

### 🚀 Running the App

```bash
docker compose up -d
```

To stop all services:

```bash
docker compose down
```

To stop and also remove the persistent volume (⚠️ deletes MongoDB data):

```bash
docker compose down -v
```

---

### 🗂️ Services Overview

| Service    | Image                    | Port Mapping  | Role                          |
|------------|--------------------------|---------------|-------------------------------|
| `frontend` | `amandhal/frontend:1.0`  | `80:80`       | React app served via Nginx    |
| `backend`  | `amandhal/backend:1.0`   | `5050:5050`   | Node.js/Express REST API      |
| `mongodb`  | `mongo:7.0-jammy`        | `27017:27017` | MongoDB database              |

---

### 🔧 docker-compose.yml Explained

```yaml
networks:
  mern_network:
    driver: bridge
```
> Creates a **custom bridge network** called `mern_network`. All three services join this network, allowing them to communicate using their **service names as hostnames** (e.g., `backend` resolves to the backend container's IP). This isolates the stack from other Docker containers on your machine.

---

```yaml
volumes:
  mongo-data:
    driver: local
```
> Declares a **named volume** `mongo-data` stored on the host machine. This ensures MongoDB data **persists across container restarts**. For example, if you run `docker compose down` and then `docker compose up` again, your database records won't be lost.

---

```yaml
  frontend:
    image: amandhal/frontend:1.0
    ports:
      - "80:80"
    networks:
      - mern_network
    environment:
      REACT_APP_API_URL: http://backend:5050
    depends_on:
      - backend
```
> - **`image`**: Pulls the pre-built React frontend image from Docker Hub.
> - **`ports: "80:80"`**: Maps port 80 on your host to port 80 inside the container. You can access the app at `http://localhost`.
> - **`environment: REACT_APP_API_URL`**: Tells the React app where to find the backend. Uses `http://backend:5050` — `backend` here is the **service name**, which Docker resolves to the backend container's IP over `mern_network`.
> - **`depends_on`**: Ensures Docker starts the `backend` container **before** the frontend. For example, without this, the frontend might boot before the API is ready.

---

```yaml
  backend:
    image: amandhal/backend:1.0
    ports:
      - "5050:5050"
    networks:
      - mern_network
    environment:
      MONGO_URI: mongodb://mongodb:27017/mern_db
    depends_on:
      - mongodb
```
> - **`ports: "5050:5050"`**: Exposes the Express API at `http://localhost:5050` on your machine.
> - **`environment: MONGO_URI`**: The connection string your Node.js app uses to connect to MongoDB. `mongodb` (lowercase) is the **service name** of the database container — Docker DNS resolves it automatically over `mern_network`. `mern_db` is the database name that will be created if it doesn't exist.
> - **`depends_on`**: Starts `mongodb` before the backend so the database is available when the API initializes.

---

```yaml
  mongodb:
    image: mongo:7.0-jammy
    ports:
      - "27017:27017"
    networks:
      - mern_network
    volumes:
      - mongo-data:/data/db
```
> - **`image: mongo:7.0-jammy`**: Uses the official MongoDB 7.0 image built on Ubuntu Jammy (22.04 LTS). Example: this is more stable and production-friendly than using `mongo:latest`.
> - **`ports: "27017:27017"`**: Exposes MongoDB locally so you can connect with tools like **MongoDB Compass** or `mongosh` from your host machine.
> - **`volumes: mongo-data:/data/db`**: Mounts the named volume to `/data/db` — the default path where MongoDB stores its data files. This is what makes your data **persistent**.

---

### 🌐 Network Communication Flow

```
Browser → localhost:80 → [frontend container]
                              ↓ REACT_APP_API_URL = http://backend:5050
                         [backend container] :5050
                              ↓ MONGO_URI = mongodb://mongodb:27017/mern_db
                         [mongodb container] :27017
```

All inter-service communication happens **inside `mern_network`** using service names, not `localhost`.


