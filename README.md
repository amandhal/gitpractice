# Container to Container Communication using different types of Docker Networks.

#### Demo 1: Test Container to Communication using Default Network.
```bash
docker run -d --name container1 -p 8081:80 -v $(pwd)/container-1-index.html:/usr/share/nginx/html/index.html nginx:alpine
docker run -d --name container2 -p 8082:80 -v $(pwd)/container-2-index.html:/usr/share/nginx/html/index.html nginx:alpine
docker ps
docker exec -it container1 curl container2
docker inspect container2 | grep -i ipaddr
docker exec -it container1 curl 172.17.0.3
docker inspect container1 | grep -i ipaddr
docker exec -it container2 curl 172.17.0.2
```
<img width="1756" height="919" alt="image" src="https://github.com/user-attachments/assets/e1daefa3-476f-4c82-bdac-43150b6a7a2d" />

------------------------------------------------------------------------

#### Demo 2: Run & Test Container on Host Network.
```bash
docker run -d --name container-on-host-network --network host -v $(pwd)/container-on-host-network-index.html:/usr/share/nginx/html/index.html nginx:alpine
docker ps
curl localhost
```
<img width="1467" height="527" alt="image" src="https://github.com/user-attachments/assets/be7966ee-159c-4c74-a178-74ba00754e7d" />

------------------------------------------------------------------------

#### Demo 3: Run & Test Container Communication on None Network.
```bash
docker run -d --name container-on-none-network --network none nginx:alpine
docker run -d --name container-on-default-network -p 8080:80 nginx:alpine
docker ps
docker inspect container-on-none-network | grep -i ipaddr
docker inspect container-on-default-network | grep -i ipaddr
docker exec -it container-on-none-network curl 172.17.0.2
curl 172.17.0.2
```
<img width="1572" height="987" alt="image" src="https://github.com/user-attachments/assets/26003abc-86b1-4a1d-bb3f-6e51a5ac23b0" />


------------------------------------------------------------------------

#### Demo 4: Create and Test Three-Tier App using custom bridge network
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
```bash
docker network create custom-bridge
docker volume create mongo-vol
docker run -d --name mongodb --network custom-bridge -p 27017:27017 -v mongo-vol:/data/db mongo:7.0-jammy
docker run -d --name backend --network custom-bridge -p 5050:5050 -e MONGO_URI="mongodb://mongodb:27017/mern_db" ghcr.io/amandhal/backend:1.0
docker run -d --name frontend --network custom-bridge -p 8080:80 ghcr.io/amandhal/frontend:1.0
docker ps
```
<img width="1919" height="1002" alt="image" src="https://github.com/user-attachments/assets/b3851aeb-07eb-427e-9c2b-e661912a1591" />
