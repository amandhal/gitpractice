# Docker Volumes - Hands-On Tasks

#### Demo 1: Named Volumes
- Create a mysql container using a named volume.
```bash
docker volume create mysql-data
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=root -v mysql-data:/var/lib/mysql mysql:8
```
<img width="1436" height="192" alt="image" src="https://github.com/user-attachments/assets/1ae28158-13a8-4fe9-82b1-f1c6ae69b633" />

---

- Create a Test Database & Insert Data into it.
```bash
docker exec -it mysql-db mysql -uroot -proot
```
```sql
-- Create a test database
CREATE DATABASE testdb;

-- Use test database and create a table
USE testdb;
CREATE TABLE users (employee_id INT, name VARCHAR(50), role VARCHAR(50));

-- Insert some rows
INSERT INTO users VALUES (1, 'Aman Dhal', 'DevOps Intern');

-- Verify inserted data
SELECT * FROM users;
```
<img width="534" height="194" alt="image" src="https://github.com/user-attachments/assets/185392f8-5ff5-4a21-942c-27068923ef5c" />

---

- Test Data Persistance by removing existing  container and then creating a new mysql container using the same volume
```bash
docker stop mysql-db
docker rm mysql-db
docker run -d --name new-mysql-db -e MYSQL_ROOT_PASSWORD=root -v mysql-data:/var/lib/mysql mysql:8
docker exec -it new-mysql-db mysql -uroot -proot
```
<img width="969" height="738" alt="image" src="https://github.com/user-attachments/assets/ab9cae20-9f2f-4061-968d-27e00c3e4754" />

---

#### Demo 2: Bind mounts
- Create an nginx container using custom index.html stored on host's filesystem
```bash
docker run -d --name nginx-bind-mount -v $(pwd)/custom-index.html:/usr/share/nginx/html/index.html nginx:alpine
docker exec -it nginx-bind-mount curl localhost
```
<img width="1641" height="454" alt="image" src="https://github.com/user-attachments/assets/d1954e68-4668-4428-bcee-fa679e1151b8" />

---

- Edit custom-index.html on host, restart same container and execute curl localhost inside container to verify if file change on host is reflected inside the container or not.
```bash
vim custom-index.html
docker restart nginx-bind-mount
docker exec -it nginx-bind-mount curl localhost
```
<img width="1379" height="450" alt="image" src="https://github.com/user-attachments/assets/35bcf6ae-235a-41fe-a794-d9354c066a17" />

---

#### Demo 3: Anonymous volumes
- Create a container using anonymous volume and add a file inside the container where the volume is mounted
```bash
docker run -d --name anonymous-container -v /anonymous-dir busybox sleep infinity
docker exec -it anonymous-container sh
```
<img width="1278" height="237" alt="image" src="https://github.com/user-attachments/assets/1ed011da-38d8-40df-9f5b-688d4d4e03eb" />
 
---

- Inspect the container to find out the exact path where anonymous volume is storing data on host and test data persistence
```bash
docker inspect anonymous-container | grep volume
docker volume ls
sudo cat /var/lib/docker/volumes/8f82fdb713c00bf29028a62dd43a36dab0c2422a6e8efd5ec4381618fc0d358c/_data/anonymous-file.txt
docker rm -f anonymous-container
sudo cat /var/lib/docker/volumes/8f82fdb713c00bf29028a62dd43a36dab0c2422a6e8efd5ec4381618fc0d358c/_data/anonymous-file.txt
```
<img width="1787" height="446" alt="image" src="https://github.com/user-attachments/assets/51d5a04b-81f9-4e61-bd39-b0b8c73ad70b" />



