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


