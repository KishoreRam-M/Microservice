Let's build a **complete microservices project** step by step with the following components:  

- **Spring Cloud Config Server** (centralized configuration)  
- **Eureka Service Registry** (service discovery)  
- **Spring Cloud Gateway** (API Gateway)  
- **User Service, Order Service, Product Service** (business microservices)  

---

# **🛠 Step 1: Setup Spring Cloud Config Server (Git-Backed)**  

## **1.1 Create Config Server Project**  
Generate a new **Spring Boot** project with dependencies:  
✅ **Spring Cloud Config Server**  
✅ **Spring Boot Actuator**  

### **1.2 Add Dependencies (`pom.xml`)**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### **1.3 Enable Config Server (`ConfigServerApplication.java`)**  
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

### **1.4 Configure Git Backend (`application.yml`)**  
```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-username/microservices-config
          clone-on-start: true
```

✅ **Git Repository Structure (`microservices-config` repo on GitHub)**  
```
microservices-config
│── user-service.yml
│── order-service.yml
│── product-service.yml
```

### **1.5 Example `user-service.yml` Configuration File**  
```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/users_db
    username: root
    password: password
```

### **1.6 Run Config Server**  
```sh
mvn spring-boot:run
```
✅ **Config Server is running at `http://localhost:8888/user-service/default`**

---

# **🛠 Step 2: Setup Eureka Service Registry**  

## **2.1 Create Eureka Server Project**  
Generate a new Spring Boot project with:  
✅ **Spring Cloud Eureka Server**  

### **2.2 Add Dependencies (`pom.xml`)**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### **2.3 Enable Eureka Server (`EurekaServerApplication.java`)**  
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### **2.4 Configure Eureka Server (`application.yml`)**  
```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
```

### **2.5 Run Eureka Server**  
```sh
mvn spring-boot:run
```
✅ **Eureka Dashboard available at `http://localhost:8761`**

---

# **🛠 Step 3: Setup API Gateway (Spring Cloud Gateway)**  

## **3.1 Create API Gateway Project**  
Generate a Spring Boot project with:  
✅ **Spring Cloud Gateway**  
✅ **Eureka Client**  

### **3.2 Add Dependencies (`pom.xml`)**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### **3.3 Configure API Gateway (`application.yml`)**  
```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/users/**
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
        - id: product-service
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/products/**

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### **3.4 Enable Eureka Client (`ApiGatewayApplication.java`)**  
```java
@SpringBootApplication
@EnableEurekaClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

### **3.5 Run API Gateway**  
```sh
mvn spring-boot:run
```
✅ **Now, all API requests go through `http://localhost:8080`**

---

# **🛠 Step 4: Create Microservices (User, Order, Product Services)**  

## **4.1 Common Configuration for Microservices (`bootstrap.yml`)**  
Each microservice should have a `bootstrap.yml` file to connect with **Config Server**.  
```yaml
spring:
  application:
    name: user-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

## **4.2 Example: User Service Implementation**  

### **Dependencies (`pom.xml`)**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### **Controller (`UserController.java`)**
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from User Service!";
    }
}
```

✅ **Similarly, create `OrderService` & `ProductService`.**  

---

# **🚀 Final Testing**  

### **1️⃣ Check Eureka Service Registry**  
- Go to `http://localhost:8761`  
- Verify that `USER-SERVICE`, `ORDER-SERVICE`, and `PRODUCT-SERVICE` are registered.

### **2️⃣ Test APIs via API Gateway**  
```sh
curl http://localhost:8080/users/hello
curl http://localhost:8080/orders/hello
curl http://localhost:8080/products/hello
```

✅ **All services should respond through the API Gateway.**  

---

# **🎯 Summary & Best Practices**  

| **Component** | **Purpose** |
|--------------|------------|
| **Config Server** | Centralized Configuration |
| **Eureka Server** | Service Discovery |
| **Spring Cloud Gateway** | API Routing & Load Balancing |
| **User, Order, Product Services** | Business Microservices |

## **Next Steps**  
Want to add **security (JWT/OAuth2), resilience (Circuit Breaker), or Kubernetes deployment**? Let me know! 🚀
