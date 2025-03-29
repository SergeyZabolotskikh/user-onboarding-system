
# User Onboarding System

## Overview

The **User Onboarding System** is a microservices-based platform designed to manage user registration, authentication, and role-based access control. The system consists of three distinct microservices:

1. **User Management Service** – Handles user registration, password updates, user deletions, and role management.
2. **Session Management Service** – Manages JWT session creation, role-based access control (RBAC), password expiry, and validates the user's security header.
3. **OAuth Service** – Provides OAuth 2.0 authentication for external clients, allowing users to log in or register using external providers like Google and Facebook. The service also supports token exchange and user session management.

The system uses **JWT tokens** for user sessions and **PostgreSQL** as the database. Kubernetes will be used for container orchestration and deployment.

---

## Tech Stack

- **Backend Framework**: Spring Boot
- **Authentication**: JWT (JSON Web Tokens)
- **OAuth 2.0**: Custom OAuth provider for user login via external providers (Google, Facebook, etc.)
- **Database**: PostgreSQL (internal with Kubernetes)
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **Security**: Spring Security for role-based access control (RBAC)
- **API Documentation**: Swagger/OpenAPI
- **Testing**: JUnit, Mockito
- **gRPC**: For inter-service communication (e.g., user suspension)

---

## Microservices Overview

### 1. User Management Service
- **Responsibilities**:
  - User registration and deletion
  - Password updates and validation
  - Role management (creating, updating, and assigning roles)
  - gRPC interface for suspending users based on inactivity
- **Endpoints**:
  - `POST /users` – Register a new user
  - `PATCH /users/{userId}/password` – Update user password
  - `DELETE /users/{userId}` – Delete a user
  - `POST /users/{userId}/roles` – Assign or update roles for a user

### 2. Session Management Service
- **Responsibilities**:
  - Creates a new session for the user using JWT
  - Validates the JWT session token
  - Verifies user roles before granting access to resources
  - Checks for password expiry and forces password changes if the password hasn’t been updated in 30 days
- **Endpoints**:
  - `POST /sessions` – Create a new session with JWT token
  - `POST /sessions/validate` – Validate an existing session and check for expiry
  - `GET /resources/{resourceId}/access` – Validate if the user has access to a resource based on roles

### 3. OAuth Service
- **Responsibilities**:
  - Provides OAuth authentication for external clients
  - Allows users to log in or register using third-party OAuth providers (Google, Facebook)
  - Manages OAuth token exchange and session creation for users
  - Issues OAuth access tokens, validates token usage, and manages user session information
- **Endpoints**:
  - `POST /oauth/authorize` – Initiate OAuth login flow
  - `POST /oauth/token` – Exchange authorization code for access token
  - `POST /oauth/refresh` – Refresh an access token using a refresh token
  - `GET /oauth/userinfo` – Retrieve user information associated with the provided access token

---

## Database Schema

### PostgreSQL
- **Tables**:
  - `users` - Stores user details (first name, last name, email, password, status, etc.)
  - `roles` - Stores roles (ADMIN, USER, etc.)
  - `role_resource_access` - Stores the resource-role mapping for access control.
  - `sessions` - stores session-related data (like JWT tokens, expiration, and user details associated with the session).

### Database Initialization Jobs
- **Kubernetes Job**: We’ll create a Kubernetes job to initialize the databases when deploying to the cluster, including setting up required tables and initial data.

---

## Deployment with Kubernetes

This system is designed to be deployed in a **Kubernetes environment** with **internal PostgreSQL**. The setup ensures that each microservice is containerized and can be scaled independently.

### Kubernetes Setup
1. **Docker Containers**: Each microservice is built as a Docker image.
2. **Kubernetes**: We use Kubernetes to manage the deployment, scaling, and orchestration of the services.
3. **PostgreSQL**: A PostgreSQL instance is deployed as an internal service within Kubernetes, and the connection strings are passed securely via Kubernetes secrets.

### Deployment Process
1. **Clone the Repository**:
    ```bash
    git clone https://github.com/yourusername/user-onboarding-system.git
    cd user-onboarding-system
    ```

2. **Build Docker Images**:
    For each microservice, use Docker to build images:
    ```bash
    docker build -t user-management-service .
    docker build -t session-management-service .
    docker build -t oauth-service .
    ```

3. **Configure Kubernetes**: Create necessary **Kubernetes configuration files** (`deployment.yaml`, `service.yaml`, etc.) for each service.
   
4. **Run Kubernetes Job**: To initialize the PostgreSQL database, we will configure a Kubernetes **job** to run the database migrations (using `Flyway` or `Liquibase`).

    Example:
    ```bash
    kubectl apply -f k8s/init-db-job.yaml
    ```

5. **Deploy the System**:
    ```bash
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
    kubectl apply -f k8s/ingress.yaml
    ```

---

## API Documentation

- The API is documented using **Swagger** and **OpenAPI** specifications.
- After deployment, you can access the API documentation at:
    ```bash
    http://localhost:8080/swagger-ui.html
    ```

---

## Testing

Unit and integration tests are written using **JUnit** and **Mockito**. To run tests:
```bash
mvn clean install
mvn test
```

---

## Contributing

1. Fork this repository
2. Create a feature branch (`git checkout -b feature-branch`)
3. Commit your changes (`git commit -m 'Add new feature'`)
4. Push to the branch (`git push origin feature-branch`)
5. Open a pull request

---
