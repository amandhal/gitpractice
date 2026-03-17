## Demo 1: Named Volumes
- #### Create a mysql container using a named volume.
```bash
docker volume create mysql-data
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=root -v mysql-data:/var/lib/mysql mysql:8
```
<img width="1436" height="192" alt="image" src="https://github.com/user-attachments/assets/1ae28158-13a8-4fe9-82b1-f1c6ae69b633" />

---

- #### Create a Test Database & Insert Data into it.
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

- #### Test Data Persistance by removing existing  container and then creating a new mysql container using the same volume
```bash
docker stop mysql-db
docker rm mysql-db
docker run -d --name new-mysql-db -e MYSQL_ROOT_PASSWORD=root -v mysql-data:/var/lib/mysql mysql:8
docker exec -it new-mysql-db mysql -uroot -proot
```
<img width="969" height="738" alt="image" src="https://github.com/user-attachments/assets/ab9cae20-9f2f-4061-968d-27e00c3e4754" />
