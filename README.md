# Contanerise & Deploy 3-Tier MERN Stack App Using Docker

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

#### Step 2: Build & Push Image to Docker Hub
```bash
docker build -t amandhal/backend:1.0 .
docker push amandhal/backend:1.0
```
