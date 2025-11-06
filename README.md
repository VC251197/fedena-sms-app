



####Backend



Repo layout (backend)

backend/
├─ docker-compose-backend.yml   # optional helper (shown below)
├─ student-service/
│  ├─ Dockerfile
│  ├─ pom.xml
│  └─ src/main/java/com/fedena/studentservice/...
├─ user-service/
│  └─ ...
├─ attendance-service/
│  └─ ...
├─ exam-service/
│  └─ ...
├─ timetable-service/
│  └─ ...
└─ finance-service/
   └─ ...


⸻

Helper docker-compose for backend (optional)

Save as docker-compose-backend.yml at repo root if you want to run all services together and point them to a MySQL container (adapt passwords/hosts as needed):

version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
    ports: ["3306:3306"]
    volumes:
      - mysql-data:/var/lib/mysql

  student-service:
    build: ./student-service
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/student_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
      - SERVER_PORT=8080
    ports: ["8080:8080"]
    depends_on: ["mysql"]

  user-service:
    build: ./user-service
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/user_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
      - SERVER_PORT=8081
    ports: ["8081:8081"]
    depends_on: ["mysql"]

  attendance-service:
    build: ./attendance-service
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/attendance_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
      - SERVER_PORT=8082
    ports: ["8082:8082"]
    depends_on: ["mysql"]

  exam-service:
    build: ./exam-service
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/exam_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
      - SERVER_PORT=8083
    ports: ["8083:8083"]
    depends_on: ["mysql"]

  timetable-service:
    build: ./timetable-service
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/timetable_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
      - SERVER_PORT=8084
    ports: ["8084:8084"]
    depends_on: ["mysql"]

  finance-service:
    build: ./finance-service
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/finance_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=rootpassword
      - SERVER_PORT=8085
    ports: ["8085:8085"]
    depends_on: ["mysql"]

volumes:
  mysql-data:


⸻

Service templates (repeat for each service, with package and DB changed)

Below I include the complete file contents for each of the six services.

⸻

1) Student Service (backend/student-service)

Dockerfile

FROM maven:3.9.3-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -B -q dependency:resolve
COPY src ./src
RUN mvn -B -q package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/student-service-0.0.1-SNAPSHOT.jar ./app.jar
EXPOSE 8080
ENTRYPOINT ["sh","-c","java -Dserver.port=${SERVER_PORT:-8080} -jar /app/app.jar"]

pom.xml

<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.fedena</groupId>
  <artifactId>student-service</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.4</version>
    <relativePath/>
  </parent>
  <dependencies>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-data-jpa</artifactId></dependency>
    <dependency><groupId>com.mysql</groupId><artifactId>mysql-connector-j</artifactId><scope>runtime</scope></dependency>
    <dependency><groupId>org.projectlombok</groupId><artifactId>lombok</artifactId><optional>true</optional></dependency>
  </dependencies>
  <build><plugins><plugin><groupId>org.springframework.boot</groupId><artifactId>spring-boot-maven-plugin</artifactId></plugin></plugins></build>
</project>

application.properties

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:mysql://localhost:3306/student_db?useSSL=false&serverTimezone=UTC}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:rootpassword}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
server.port=${SERVER_PORT:8080}
management.endpoints.web.exposure.include=health,info

Java sources (package com.fedena.studentservice)

StudentServiceApplication.java

package com.fedena.studentservice;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class StudentServiceApplication {
  public static void main(String[] args) { SpringApplication.run(StudentServiceApplication.class, args); }
}

model/Student.java

package com.fedena.studentservice.model;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name="students")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor @ToString
public class Student {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private String name;
  private String className;
  private Integer roll;
}

repo/StudentRepository.java

package com.fedena.studentservice.repo;
import org.springframework.data.jpa.repository.JpaRepository;
import com.fedena.studentservice.model.Student;
public interface StudentRepository extends JpaRepository<Student, Long> {}

controller/StudentController.java

package com.fedena.studentservice.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.fedena.studentservice.model.Student;
import com.fedena.studentservice.repo.StudentRepository;

@RestController
@RequestMapping("/api/students")
@CrossOrigin(origins = "*")
public class StudentController {
  @Autowired private StudentRepository repo;
  @GetMapping public List<Student> list(){ return repo.findAll(); }
  @PostMapping public Student create(@RequestBody Student s){ return repo.save(s); }
  @GetMapping("/{id}") public Student get(@PathVariable Long id){ return repo.findById(id).orElseThrow(); }
  @DeleteMapping("/{id}") public void delete(@PathVariable Long id){ repo.deleteById(id); }
}


⸻

2) User Service (backend/user-service)

Dockerfile

FROM maven:3.9.3-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -B -q dependency:resolve
COPY src ./src
RUN mvn -B -q package -DskipTests
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/user-service-0.0.1-SNAPSHOT.jar ./app.jar
EXPOSE 8081
ENTRYPOINT ["sh","-c","java -Dserver.port=${SERVER_PORT:-8081} -jar /app/app.jar"]

pom.xml
(same dependencies, artifactId user-service)

application.properties

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:mysql://localhost:3306/user_db?useSSL=false&serverTimezone=UTC}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:rootpassword}
spring.jpa.hibernate.ddl-auto=update
server.port=${SERVER_PORT:8081}
management.endpoints.web.exposure.include=health,info

Java (package com.fedena.userservice)

UserServiceApplication.java

package com.fedena.userservice;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class UserServiceApplication {
  public static void main(String[] args){ SpringApplication.run(UserServiceApplication.class, args); }
}

model/User.java

package com.fedena.userservice.model;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name="users")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class User {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private String name;
  private String email;
  private String role;
}

repo/UserRepository.java

package com.fedena.userservice.repo;
import org.springframework.data.jpa.repository.JpaRepository;
import com.fedena.userservice.model.User;
public interface UserRepository extends JpaRepository<User, Long> {}

controller/UserController.java

package com.fedena.userservice.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.fedena.userservice.model.User;
import com.fedena.userservice.repo.UserRepository;

@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "*")
public class UserController {
  @Autowired private UserRepository repo;
  @GetMapping public List<User> list(){ return repo.findAll(); }
  @PostMapping public User create(@RequestBody User u){ return repo.save(u); }
  @GetMapping("/{id}") public User get(@PathVariable Long id){ return repo.findById(id).orElseThrow(); }
  @DeleteMapping("/{id}") public void delete(@PathVariable Long id){ repo.deleteById(id); }
}


⸻

3) Attendance Service (backend/attendance-service)

Dockerfile — similar (expose 8082)

application.properties

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:mysql://localhost:3306/attendance_db?useSSL=false&serverTimezone=UTC}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:rootpassword}
spring.jpa.hibernate.ddl-auto=update
server.port=${SERVER_PORT:8082}
management.endpoints.web.exposure.include=health,info

Java (package com.fedena.attendanceservice)

AttendanceServiceApplication.java

package com.fedena.attendanceservice;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class AttendanceServiceApplication {
  public static void main(String[] args){ SpringApplication.run(AttendanceServiceApplication.class, args); }
}

model/Attendance.java

package com.fedena.attendanceservice.model;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name="attendance")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Attendance {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private Long studentId;
  private String date;
  private Boolean present;
}

repo/AttendanceRepository.java

package com.fedena.attendanceservice.repo;
import org.springframework.data.jpa.repository.JpaRepository;
import com.fedena.attendanceservice.model.Attendance;
import java.util.List;
public interface AttendanceRepository extends JpaRepository<Attendance, Long>{
  List<Attendance> findByStudentId(Long studentId);
}

controller/AttendanceController.java

package com.fedena.attendanceservice.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.fedena.attendanceservice.model.Attendance;
import com.fedena.attendanceservice.repo.AttendanceRepository;

@RestController
@RequestMapping("/api/attendance")
@CrossOrigin(origins = "*")
public class AttendanceController {
  @Autowired private AttendanceRepository repo;
  @GetMapping public List<Attendance> list(){ return repo.findAll(); }
  @GetMapping("/student/{sid}") public List<Attendance> byStudent(@PathVariable Long sid){ return repo.findByStudentId(sid); }
  @PostMapping public Attendance create(@RequestBody Attendance a){ return repo.save(a); }
}


⸻

4) Exam Service (backend/exam-service)

application.properties (port 8083)

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:mysql://localhost:3306/exam_db?useSSL=false&serverTimezone=UTC}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:rootpassword}
spring.jpa.hibernate.ddl-auto=update
server.port=${SERVER_PORT:8083}
management.endpoints.web.exposure.include=health,info

Java (package com.fedena.examservice)

ExamServiceApplication.java

package com.fedena.examservice;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class ExamServiceApplication {
  public static void main(String[] args){ SpringApplication.run(ExamServiceApplication.class, args); }
}

model/Exam.java

package com.fedena.examservice.model;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name="exams")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Exam {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private String title;
  private String date;
  private String subject;
  private Integer maxMarks;
}

repo/ExamRepository.java

package com.fedena.examservice.repo;
import org.springframework.data.jpa.repository.JpaRepository;
import com.fedena.examservice.model.Exam;
public interface ExamRepository extends JpaRepository<Exam, Long> {}

controller/ExamController.java

package com.fedena.examservice.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.fedena.examservice.model.Exam;
import com.fedena.examservice.repo.ExamRepository;

@RestController
@RequestMapping("/api/exams")
@CrossOrigin(origins = "*")
public class ExamController {
  @Autowired private ExamRepository repo;
  @GetMapping public List<Exam> list(){ return repo.findAll(); }
  @PostMapping public Exam create(@RequestBody Exam e){ return repo.save(e); }
}


⸻

5) Timetable Service (backend/timetable-service)

application.properties (port 8084)

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:mysql://localhost:3306/timetable_db?useSSL=false&serverTimezone=UTC}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:rootpassword}
spring.jpa.hibernate.ddl-auto=update
server.port=${SERVER_PORT:8084}
management.endpoints.web.exposure.include=health,info

Java (package com.fedena.timetableservice)

TimetableServiceApplication.java

package com.fedena.timetableservice;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class TimetableServiceApplication {
  public static void main(String[] args){ SpringApplication.run(TimetableServiceApplication.class, args); }
}

model/Timetable.java

package com.fedena.timetableservice.model;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name="timetable")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Timetable {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private String className;
  private String day;
  @Column(columnDefinition = "TEXT")
  private String scheduleJson;
}

repo/TimetableRepository.java

package com.fedena.timetableservice.repo;
import org.springframework.data.jpa.repository.JpaRepository;
import com.fedena.timetableservice.model.Timetable;
import java.util.List;
public interface TimetableRepository extends JpaRepository<Timetable, Long>{
  List<Timetable> findByClassName(String className);
}

controller/TimetableController.java

package com.fedena.timetableservice.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.fedena.timetableservice.model.Timetable;
import com.fedena.timetableservice.repo.TimetableRepository;

@RestController
@RequestMapping("/api/timetable")
@CrossOrigin(origins = "*")
public class TimetableController {
  @Autowired private TimetableRepository repo;
  @GetMapping public List<Timetable> list(){ return repo.findAll(); }
  @GetMapping("/class/{c}") public List<Timetable> byClass(@PathVariable String c){ return repo.findByClassName(c); }
  @PostMapping public Timetable create(@RequestBody Timetable t){ return repo.save(t); }
}


⸻

6) Finance Service (backend/finance-service)

application.properties (port 8085)

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:mysql://localhost:3306/finance_db?useSSL=false&serverTimezone=UTC}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:root}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:rootpassword}
spring.jpa.hibernate.ddl-auto=update
server.port=${SERVER_PORT:8085}
management.endpoints.web.exposure.include=health,info

Java (package com.fedena.financeservice)

FinanceServiceApplication.java

package com.fedena.financeservice;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class FinanceServiceApplication {
  public static void main(String[] args){ SpringApplication.run(FinanceServiceApplication.class, args); }
}

model/Invoice.java

package com.fedena.financeservice.model;
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name="invoices")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Invoice {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private Long studentId;
  private Double amount;
  private String dueDate;
  private Boolean paid;
}

repo/InvoiceRepository.java

package com.fedena.financeservice.repo;
import org.springframework.data.jpa.repository.JpaRepository;
import com.fedena.financeservice.model.Invoice;
import java.util.List;
public interface InvoiceRepository extends JpaRepository<Invoice, Long>{
  List<Invoice> findByStudentId(Long studentId);
}

controller/InvoiceController.java

package com.fedena.financeservice.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.fedena.financeservice.model.Invoice;
import com.fedena.financeservice.repo.InvoiceRepository;

@RestController
@RequestMapping("/api/invoices")
@CrossOrigin(origins = "*")
public class InvoiceController {
  @Autowired private InvoiceRepository repo;
  @GetMapping public List<Invoice> list(){ return repo.findAll(); }
  @GetMapping("/student/{sid}") public List<Invoice> byStudent(@PathVariable Long sid){ return repo.findByStudentId(sid); }
  @PostMapping public Invoice create(@RequestBody Invoice i){ return repo.save(i); }
}


⸻

How to build & run (local / cloud)

Option A — Build with Maven & run locally

For each service folder:

cd backend/student-service
mvn clean package -DskipTests
java -jar target/student-service-0.0.1-SNAPSHOT.jar

Make sure application.properties DB URL points to a running MySQL and DB exists. For dev, you can create DBs manually:

CREATE DATABASE student_db;
CREATE DATABASE user_db;
CREATE DATABASE attendance_db;
CREATE DATABASE exam_db;
CREATE DATABASE timetable_db;
CREATE DATABASE finance_db;

Option B — Use Docker (recommended for cloud)
	1.	Ensure MySQL is running (container or managed). If you want to run MySQL container:

docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=rootpassword -p 3306:3306 mysql:8.0
# create DBs (connect with mysql client and run CREATE DATABASE ...)

	2.	Build & run a service image:

cd backend/student-service
docker build -t fedena/student-service:latest .
docker run -d --name student-service -p 8080:8080 \
  -e SPRING_DATASOURCE_URL="jdbc:mysql://host.docker.internal:3306/student_db?useSSL=false&serverTimezone=UTC" \
  -e SPRING_DATASOURCE_USERNAME=root \
  -e SPRING_DATASOURCE_PASSWORD=rootpassword \
  -e SERVER_PORT=8080 \
  fedena/student-service:latest

(If you’re running everything via the docker-compose-backend.yml, use that to wire services and MySQL automatically.)

Option C — Docker Compose (one command)

If you used docker-compose-backend.yml earlier, place it at backend root and run:

docker compose -f docker-compose-backend.yml up --build -d


⸻

Quick sanity checks & troubleshooting
	•	If a service fails to start: check logs docker logs <container> or mvn spring-boot:run output.
	•	DB connection errors: verify MySQL host/port, user/password, and that DB exists.
	•	Port conflicts: ensure ports 8080–8085 are free.
	•	JPA not creating tables: ensure spring.jpa.hibernate.ddl-auto=update and that connection is successful.
	•	Use curl http://localhost:8080/actuator/health (if actuator included) to check health.

⸻