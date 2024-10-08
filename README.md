# asc-cms-server

This is an open-source backend server for a Course Management System (CMS). The web client can be found [here](https://github.com/s-bose7/asc-cms-web-client). The application strictly follows [MVC](docs/asc-mvc-architecture.png) architectural pattern, and utilizes RESTful APIs for exchanging information between the server and client. 

# Prerequisite

- Make sure you have Docker installed on your machine.
- Make sure you have Java17 installed on your machine.

# Build Process

## Running locally
```bash
# Get the code
git clone https://github.com/s-bose7/asc-cms-api.git

# Move to project folder
cd asc-cms-api

# Build the application
./gradlew clean build

# Run the server
./gradlew bootRun
```
After running the server one can test the APIs hosted at `http://localhost:8080`. For more infomation on what kind of requests are allowed [see here](docs/api-docs.md).

## Using Docker compose
Using docker compose to start the application. 

```bash
$ docker compose up -d
[+] Running 3/3
 ✔ Container asc-cms-api-db         Started          0.4s 
 ✔ Container asc-cms-api-server-1   Started          0.8s 
 ✔ Container asc-cms-api-client-1   Started          1.3s 

```
Once all the containers are up and running, visit `http://localhost:5173`. This will take you to the web client.

# Example Requests
The payloads are case sensitive, so please double-check the payload before sending a request.

#### Creating a Course

```c
curl -X POST http://localhost:8080/api/v1/courses \
-header "Content-Type: application/json" \
-data '{
    "courseTitle": "Introduction to Compiler Design",
    "courseCode": "CS 101",
    "courseDescription": "This course provides an introduction to modern compilers."
}'

```

#### Creating a Course instance

A course instance or course delivery is created primarily based on the course code, though the actual JSON payload is different. First the client request the server with a course code, year and semester, the server verifies the course code,
if valid, then the web client will sends a `POST` request with the following payload as request body. Otherwise the requests fails.

```c
curl -X POST http://localhost:8080/api/v1/instances \
-header "Content-Type: application/json" \
-data '{
    "year": 2024,
    "semester": 5,
    "course": {
        "id": 2,
        "courseTitle": "Introduction to Compiler Design",
        "courseCode": "CS 101",
        "courseDescription": "This course provides an introduction to modern compilers",
        "createdAt": "2024-08-24"
    }
}'
```

# Database schema overview

```
# Primary table to store information about courses

mysql> describe courses; 
+--------------------+--------------+------+-----+---------+----------------+
| Field              | Type         | Null | Key | Default | Extra          |
+--------------------+--------------+------+-----+---------+----------------+
| id                 | bigint       | NO   | PRI | NULL    | auto_increment |
| course_code        | varchar(10)  | NO   | UNI | NULL    |                |
| course_description | varchar(300) | YES  |     | NULL    |                |
| course_title       | varchar(60)  | NO   |     | NULL    |                |
| created_at         | date         | NO   |     | NULL    |                |
+--------------------+--------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```
**Relation**: One to Many (1:N); A course can be part of many delivery.

```
# Table to store inforemation about course sessions

mysql> describe session;
+----------------------+--------+------+-----+---------+----------------+
| Field                | Type   | Null | Key | Default | Extra          |
+----------------------+--------+------+-----+---------+----------------+
| id                   | bigint | NO   | PRI | NULL    | auto_increment |
| course_id            | bigint | NO   | MUL | NULL    |                |
| year_of_delivery     | int    | NO   |     | NULL    |                |
| semester_of_delivery | int    | NO   |     | NULL    |                |
+----------------------+--------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```