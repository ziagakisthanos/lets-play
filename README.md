# lets-play

A RESTful CRUD API built with Spring Boot and MongoDB Atlas. It manages
**users** and **products**, secured with JWT authentication, role-based
authorization, bcrypt password hashing, and HTTPS.

## Tech stack

- Java 17, Spring Boot 3.2.5
- Spring Web, Spring Security, Spring Data MongoDB
- MongoDB Atlas
- JWT (jjwt 0.11.5)
- Bean Validation, Lombok
- Maven (wrapper included)
- Mockito, JUnit for Testing

## Getting started

### Prerequisites

- JDK 17+
- A MongoDB Atlas cluster (connection string)
- A PKCS12 keystore on the classpath (`src/main/resources/keystore.p12`) for HTTPS

### Configuration

Copy the example config and fill in your values:

```bash
cp application.properties.example src/main/resources/application.properties
```

| Property | Description |
|---|---|
| `spring.data.mongodb.uri` | MongoDB Atlas connection string |
| `jwt.secret` | 256-bit secret for signing tokens |
| `jwt.expiration` | Token lifetime in ms (e.g. `3600000`) |
| `server.port` | HTTPS port (e.g. `8443`) |
| `server.ssl.*` | Keystore location, password, type, alias |

### Run

```bash
./mvnw spring-boot:run
```

The API is served at `https://localhost:8443`.

On first startup an admin user is seeded:

- **email:** `admin@email.com`
- **password:** `admin123`

## Authentication

1. `POST /auth/register` or log in as the seeded admin.
2. `POST /auth/login` returns a JWT.
3. Send it on protected requests: `Authorization: Bearer <token>`.

## Endpoints

### Auth

| Method | Path | Access | Description |
|---|---|---|---|
| POST | `/auth/register` | Public | Register a user (role `USER`) |
| POST | `/auth/login` | Public | Obtain a JWT |

### Products

| Method | Path | Access | Description |
|---|---|---|---|
| GET | `/products` | Public | List all products |
| GET | `/products/{id}` | Public | Get one product |
| POST | `/products` | Authenticated | Create a product (owned by caller) |
| PUT | `/products/{id}` | Owner only | Update own product |
| DELETE | `/products/{id}` | Owner only | Delete own product |

### Users

| Method | Path | Access | Description |
|---|---|---|---|
| GET | `/users` | `ADMIN` | List all users |
| DELETE | `/users/{id}` | `ADMIN` | Delete a user |

## Data models

**User** — `id`, `name`, `email` (unique), `password` (bcrypt, never serialized), `role` (`ADMIN` \| `USER`)

**Product** — `id`, `name`, `description`, `price` (> 0), `userId` (owner)

## Validation & errors

Requests are validated (non-blank name, valid email, password 6–20 chars,
positive price). Errors return a consistent JSON body:

```json
{
  "status": 404,
  "error": "Not Found",
  "message": "Product with id: 123 not found",
  "path": "/products/123"
}
```

| Status | When |
|---|---|
| 400 | Validation failure |
| 401 | Invalid credentials / missing token |
| 403 | Modifying a product you don't own |
| 404 | User or product not found |
| 409 | Email already registered |

## Tests

```bash
./mvnw test
```

## Project layout

```
controller/  REST endpoints
service/     business logic
repository/   Spring Data Mongo repositories
entity/       Mongo documents
dto/          request/response payloads
security/     JWT filter & service
config/       security and CORS configuration
exception/    custom exceptions + global handler
seeder/       admin bootstrap
```
