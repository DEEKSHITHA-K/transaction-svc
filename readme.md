# Transaction Service

A Spring Boot REST API for managing and querying financial transactions.

## Overview

A lightweight microservice that provides endpoints to retrieve transaction data using Spring Boot, Spring Data JPA, and an H2 in-memory database. Built as part of the OpenAI-with-Java learning project.

## Tech Stack

- **Language**: Java 17
- **Framework**: Spring Boot 4.0.5
- **Data Access**: Spring Data JPA
- **Database**: H2 (in-memory)
- **Build Tool**: Maven
- **Dependencies**: Lombok, Spring AI 2.0.0-M3

## Project Structure
```
transaction-svc/
├── src/main/
│   ├── java/com/example/demo/
│   │   ├── DemoApplication.java           # Spring Boot entry point
│   │   ├── Transaction.java               # JPA Entity
│   │   ├── TransactionController.java     # REST Controller
│   │   ├── TransactionService.java        # Business Logic
│   │   └── TransactionRepository.java     # Data Access Layer (JPA)
│   └── resources/
│       └── application.properties         # Application Configuration
├── pom.xml                                # Maven configuration
├── mvnw / mvnw.cmd                        # Maven wrapper
└── README.md                              # This file
```
## Getting Started

### Prerequisites

- Java 17 or higher
- Maven 3.6 or higher

### Build

```bash
mvn clean install
```

### Run

```bash
mvn spring-boot:run
```

The application starts on **port 8081** → http://localhost:8081

## API Endpoints

### 1. Get Last 5 Transactions

```http
GET /transactions/last5?userId={userId}
```

**Description**: Retrieves the 5 most recent transactions for a user, sorted by timestamp (newest first).

**Query Parameters**:
- `userId` (required): User identifier string

**Example Request**:
```bash
curl "http://localhost:8081/transactions/last5?userId=user123"
```

**Example Response**:
```json
[
  {
    "id": 1,
    "userId": "user123",
    "amount": 150.50,
    "type": "CREDIT",
    "timestamp": "2026-06-30T10:30:00"
  },
  {
    "id": 2,
    "userId": "user123",
    "amount": 75.25,
    "type": "DEBIT",
    "timestamp": "2026-06-29T14:15:00"
  }
]
```

---

### 2. Get Highest Transaction

```http
GET /transactions/highest?userId={userId}
```

**Description**: Retrieves the transaction with the highest amount for a user.

**Query Parameters**:
- `userId` (required): User identifier string

**Example Request**:
```bash
curl "http://localhost:8081/transactions/highest?userId=user123"
```

**Example Response**:
```json
{
  "id": 5,
  "userId": "user123",
  "amount": 500.00,
  "type": "CREDIT",
  "timestamp": "2026-06-25T14:22:00"
}
```

---

## Entity Model

### Transaction

| Field | Type | Description |
|-------|------|-------------|
| `id` | Long | Auto-generated primary key |
| `userId` | String | User identifier |
| `amount` | Double | Transaction amount |
| `type` | String | Transaction type: `CREDIT` or `DEBIT` |
| `timestamp` | LocalDateTime | Transaction date and time |

---

## Database Configuration

**Type**: H2 In-Memory Database

### Connection Details

| Property | Value |
|----------|-------|
| **JDBC URL** | jdbc:h2:mem:txdb |
| **Driver Class** | org.h2.Driver |
| **Username** | sa |
| **Password** | (empty) |
| **DDL Auto** | create |

### H2 Web Console

Access the H2 database browser:

- **URL**: http://localhost:8081/h2-console
- **JDBC URL**: jdbc:h2:mem:txdb
- **Username**: sa
- **Password**: (leave blank)
- **Driver Class**: org.h2.Driver

### Important Notes

- Data is **recreated on each application restart** (in-memory database)
- Schema is auto-created from entity annotations (`@Entity`, `@Id`, etc.)
- To persist data, switch to a file-based or external database (PostgreSQL, MySQL, etc.)

---

## Application Configuration

File: `src/main/resources/application.properties`

```properties
# Application Name
spring.application.name=demo

# Server Configuration
server.port=8081

# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:txdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA / Hibernate Configuration
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true

# H2 Console Configuration
spring.h2.console.enabled=true
```

### Configuration Properties Explained

| Property | Value | Purpose |
|----------|-------|---------|
| `server.port` | 8081 | Server port number |
| `spring.datasource.url` | jdbc:h2:mem:txdb | H2 in-memory database connection URL |
| `spring.datasource.driverClassName` | org.h2.Driver | H2 database driver class |
| `spring.datasource.username` | sa | Database username |
| `spring.datasource.password` | (empty) | Database password |
| `spring.jpa.hibernate.ddl-auto` | create | Auto-create schema on startup |
| `spring.jpa.show-sql` | true | Log SQL statements in console |
| `spring.h2.console.enabled` | true | Enable H2 web console |

---

## Architecture

```
┌─────────────────────────────────────────┐
│      HTTP Request (REST Client)         │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   TransactionController (REST)          │
│   - @RestController                     │
│   - @RequestMapping("/transactions")    │
│   - /last5, /highest endpoints          │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   TransactionService (Business Logic)   │
│   - getLast5(userId)                    │
│   - getHighest(userId)                  │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   TransactionRepository (Data Access)   │
│   - JpaRepository<Transaction, Long>    │
│   - findTop5ByUserIdOrderByTimestampDesc│
│   - findTopByUserIdOrderByAmountDesc    │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   Transaction Entity (JPA Model)        │
│   - id, userId, amount, type, timestamp │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│   H2 In-Memory Database                 │
│   - jdbc:h2:mem:txdb                    │
└─────────────────────────────────────────┘
```

---

## Dependencies

### Core Dependencies

```xml
<!-- Web & REST API -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Data Access & ORM -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- In-Memory Database -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- Lombok - Reduce Boilerplate Code -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

### Versions

- **Spring Boot**: 4.0.5
- **Java**: 17
- **Spring AI BOM**: 2.0.0-M3 (for future AI integrations)

---

## Key Features

✅ RESTful API for transaction queries  
✅ Spring Data JPA with custom query methods  
✅ H2 in-memory database (zero configuration)  
✅ Lombok for reduced boilerplate code  
✅ Automatic schema creation from JPA entities  
✅ H2 web console for database inspection  
✅ SQL logging in console for debugging  
✅ Spring AI integration ready

---

## Usage Examples

### Using curl

**Get last 5 transactions for user123**:
```bash
curl "http://localhost:8081/transactions/last5?userId=user123"
```

**Get highest transaction for user123**:
```bash
curl "http://localhost:8081/transactions/highest?userId=user123"
```

### Using Postman

1. **Import URLs**:
    - `GET http://localhost:8081/transactions/last5?userId=user123`
    - `GET http://localhost:8081/transactions/highest?userId=user123`

2. Set `userId` query parameter and send request

---

## Troubleshooting

### Port 8081 already in use

Change the port in `application.properties`:
```properties
server.port=8082
```

### H2 Console not accessible

Ensure `spring.h2.console.enabled=true` in `application.properties`

### No data in database

H2 in-memory database is recreated on each restart. To persist data:
- Switch to file-based H2: `spring.datasource.url=jdbc:h2:~/txdb`
- Or use external database (PostgreSQL, MySQL)

### SQL queries not showing in console

Enable SQL logging in `application.properties`:
```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

---

## Future Enhancements

- ✨ OpenAI integration for transaction analysis
- ➕ Add transaction creation/update endpoints (POST, PUT, DELETE)
- 💾 Persistent database support (PostgreSQL, MySQL)
- 🔐 Authentication & authorization (Spring Security)
- 📄 Pagination and advanced filtering
- 🧪 Unit & integration tests
- 🐳 Docker containerization
- 📚 API documentation (Swagger/OpenAPI)
- 🔔 Transaction notifications
- 📊 Analytics and reporting endpoints

---



## Related Project

Part of the **OpenAI-with-Java** learning project, which demonstrates how to integrate OpenAI's GPT models with Java applications using Spring Boot and Spring AI.


For more information, visit the parent project directory. [OpenAI-with-Java](https://github.com/DEEKSHITHA-K/OpenAI-with-java)

For the Spring Boot microservice that bridges natural language with API calls using OpenAI's GPT models.[User Prmpt to API Service](https://github.com/DEEKSHITHA-K/user-prompt-to-api-svc)

---

## Support

For issues or questions, refer to the project documentation or Spring Boot official documentation:
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [H2 Database](https://www.h2database.com)


