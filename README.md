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

#### Step 2: Create Dockerfile for Frontend Tier
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

#### Step 3: Build & Push Backend Image to GHCR
```bash
docker build -t ghcr.io/amandhal/backend:1.0 .
docker push ghcr.io/amandhal/backend:1.0
```
<img width="1558" height="689" alt="Image" src="https://github.com/user-attachments/assets/a32322f4-ccab-400b-9042-ebb6cc8aa6c9" />

#### Step 4: Build & Push Frontend Image to GHCR
```bash
docker build -t ghcr.io/amandhal/frontend:1.0 .
docker push ghcr.io/amandhal/frontend:1.0
```
<img width="1550" height="685" alt="Image" src="https://github.com/user-attachments/assets/6796a255-2776-4afa-8a0d-846fb2ea8e40" />
<img width="1883" height="636" alt="Image" src="https://github.com/user-attachments/assets/6d62c13d-5abf-4330-ba96-ebf0edd94030" />

#### Step 5: Build & Push Backend Image to Docker Hub
```bash
docker build -t amandhal/backend:1.0 .
docker push amandhal/backend:1.0
```
<img width="1521" height="655" alt="image" src="https://github.com/user-attachments/assets/c2642410-6faf-4bac-a5ec-330284baa879" />

#### Step 6: Build & Push Frontend Image to Docker Hub
```bash
docker build -t amandhal/frontend:1.0 .
docker push amandhal/frontend:1.0
```
<img width="1540" height="687" alt="image" src="https://github.com/user-attachments/assets/a091bbc6-e2d2-44a2-b20c-c184686bcdbc" />
<img width="1902" height="617" alt="Image" src="https://github.com/user-attachments/assets/f5d367a2-0f57-4146-9761-45579509d05d" />

#### Step 7: Create Docker Compose File
```yaml
services:
  frontend:
    image: amandhal/frontend:1.0
    ports:
      - "80:80"  
    networks:
      - mern_network
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

#### Step 8: Deploy App Using Docker Compose
```bash
docker compose up -d
```
<img width="1852" height="481" alt="image" src="https://github.com/user-attachments/assets/6844036e-59f6-4c2e-b453-4b2474777e12" />
<img width="1843" height="373" alt="image" src="https://github.com/user-attachments/assets/4bf31a1e-621e-4192-a75a-78cdc5b75ff2" />

