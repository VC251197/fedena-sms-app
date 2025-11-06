

⸻


#### 🎓 Fedena School Management System — Backend

This repository contains the *backend microservices* for the *Fedena School Management System clone*, built using:

- *Spring Boot 3.2*
- *Java 17*
- *Maven*
- *MySQL*
- *Docker / Docker Compose*
- (DevOps ready for AWS EC2, Jenkins, and Kubernetes deployment)

---

## 🏗 Architecture Overview

The backend is designed as *six independent microservices*, each managing a specific domain of the school management system.  
All services are RESTful and connect to their respective MySQL databases.

| Service Name         | Port  | Database Name     | Description |
|----------------------|-------|-------------------|-------------|
| student-service    | 8080  | student_db      | Manages student information |
| user-service       | 8081  | user_db         | Handles staff and admin users |
| attendance-service | 8082  | attendance_db   | Records daily attendance |
| exam-service       | 8083  | exam_db         | Stores exams and marks |
| timetable-service  | 8084  | timetable_db    | Manages class schedules |
| finance-service    | 8085  | finance_db      | Handles fees and invoices |

Each microservice runs *independently* and exposes REST APIs for the frontend.

---

## 📂 Project Structure

backend/
├── docker-compose-backend.yml
├── student-service/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── user-service/
├── attendance-service/
├── exam-service/
├── timetable-service/
└── finance-service/

---

## 🚀 Quick Start (Local Setup)

### 🧩 1. Prerequisites
Make sure you have installed:
- [Java 17+](https://adoptium.net/)
- [Maven 3.9+](https://maven.apache.org/)
- [MySQL 8+](https://www.mysql.com/)
- [Docker](https://www.docker.com/) (optional but recommended)

---

### ⚙ 2. Database Setup

Create six databases manually in MySQL or via a script:

```sql
CREATE DATABASE student_db;
CREATE DATABASE user_db;
CREATE DATABASE attendance_db;
CREATE DATABASE exam_db;
CREATE DATABASE timetable_db;
CREATE DATABASE finance_db;

You can also use Docker Compose (see below) to create the MySQL container automatically.

⸻

🧪 3. Run Individual Services (Without Docker)

For each service:

cd backend/student-service
mvn clean package -DskipTests
java -jar target/student-service-0.0.1-SNAPSHOT.jar

Repeat for all services (ports 8080–8085).
Each service connects to its respective database as configured in application.properties.

⸻

🐳 4. Run All Services with Docker Compose

Simply run all services and MySQL using the included Docker Compose file:

cd backend
docker compose -f docker-compose-backend.yml up --build -d

This command will:
	•	Start a MySQL container
	•	Build and run all six microservices
	•	Expose ports 8080–8085

To check logs:

docker compose logs -f student-service

To stop:

docker compose down


⸻

🔍 API Overview

Each service exposes a RESTful API on its port.

Service	Example Endpoint	Method	Description
Student Service	/api/students	GET	Get all students
	/api/students	POST	Add new student
	/api/students/{id}	GET	Get a specific student
User Service	/api/users	GET/POST	Manage users
Attendance Service	/api/attendance/student/{id}	GET	Get attendance by student
Exam Service	/api/exams	GET/POST	Manage exams
Timetable Service	/api/timetable/class/{name}	GET	Get timetable by class
Finance Service	/api/invoices/student/{id}	GET	Get invoices for student

All endpoints return JSON responses and accept JSON bodies.

⸻

🧰 Environment Variables

Each service reads its configuration from environment variables (with defaults):

Variable	Default	Description
SPRING_DATASOURCE_URL	jdbc:mysql://localhost:3306/<db>	MySQL connection URL
SPRING_DATASOURCE_USERNAME	root	MySQL username
SPRING_DATASOURCE_PASSWORD	rootpassword	MySQL password
SERVER_PORT	(service specific)	Port for the service

Example:

SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/student_db \
SPRING_DATASOURCE_USERNAME=root \
SPRING_DATASOURCE_PASSWORD=rootpassword \
SERVER_PORT=8080


⸻

☁ DevOps & Cloud Deployment

You can easily deploy this backend on cloud or Kubernetes:

AWS EC2 + Docker

# Build image
docker build -t fedena/student-service:latest backend/student-service
# Run container
docker run -d -p 8080:8080 fedena/student-service:latest

Jenkins CI/CD Pipeline (example)
	•	Step 1: Clone repo
	•	Step 2: mvn clean package -DskipTests
	•	Step 3: Build & push Docker image to ECR / DockerHub
	•	Step 4: Deploy to EC2 or Kubernetes

Kubernetes Deployment (basic example)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: student-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: student-service
  template:
    metadata:
      labels:
        app: student-service
    spec:
      containers:
      - name: student-service
        image: fedena/student-service:latest
        ports:
        - containerPort: 8080


⸻

🧠 Tech Stack Summary

Layer	Technology
Backend Framework	Spring Boot (Java 17)
Build Tool	Maven
Database	MySQL
Containerization	Docker
Service Management	Docker Compose / Kubernetes
Deployment	AWS / Cloud-ready
Monitoring	Spring Boot Actuator
Logging	SLF4J / Logback


⸻

🧑‍💻 Developer Notes
	•	Each service is stateless — scaling horizontally is easy.
	•	Default Hibernate strategy: ddl-auto=update.
	•	For production, configure:
	•	Centralized MySQL (RDS or managed DB)
	•	Centralized logging (ELK, CloudWatch)
	•	API Gateway / Service Discovery (Spring Cloud Netflix / Nginx)

⸻

✅ Health Check

You can verify if a service is running:

curl http://localhost:8080/actuator/health

Expected output:

{"status":"UP"}


⸻

🧾 License

This project is open-source under the MIT License￼.

⸻

💡 Next Steps
	•	Add auth-service with JWT Authentication
	•	Integrate all APIs into the React frontend
	•	Add CI/CD pipeline with Jenkins or GitHub Actions
	•	Deploy full stack to AWS (EC2 or EKS)

⸻



