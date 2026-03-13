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

#### Step 5: Deploy MERN APP Using Docker Compose
```yaml
services:
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

  mongodb:
    image: mongo:7.0-jammy 
    ports:
      - "27017:27017"  
    networks:
      - mern_network
    volumes:
      - mongo-data:/data/db

networks:
  mern_network:
    driver: bridge 

volumes:
  mongo-data:
    driver: local
  ```
```bash
docker compose up -d
```
