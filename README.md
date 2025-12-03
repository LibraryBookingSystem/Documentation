# Library System - Getting Started Guide

This guide will help you set up and run the Library System microservices application.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Architecture Overview](#architecture-overview)
- [Running the Application](#running-the-application)
  - [Option 1: Docker Compose (Recommended)](#option-1-docker-compose-recommended)
  - [Option 2: Individual Service Build](#option-2-individual-service-build)
  - [Option 3: Local Development](#option-3-local-development)
- [Accessing Services](#accessing-services)
- [Verifying the Setup](#verifying-the-setup)
- [Common Operations](#common-operations)
- [Troubleshooting](#troubleshooting)
- [Development Workflow](#development-workflow)

---

## Prerequisites

Before running the application, ensure you have the following installed:

### Required Software

1. **Docker** (version 20.10 or higher)
   - Download from: https://www.docker.com/get-started
   - Verify installation: `docker --version`

2. **Docker Compose** (version 2.0 or higher)
   - Usually included with Docker Desktop
   - Verify installation: `docker-compose --version`

3. **Java 17** (for local development)
   - Download from: https://adoptium.net/
   - Verify installation: `java -version`

4. **Maven 3.9+** (for local development)
   - Download from: https://maven.apache.org/download.cgi
   - Verify installation: `mvn --version`

### System Requirements

- **RAM**: Minimum 8GB (16GB recommended)
- **Disk Space**: At least 10GB free
- **Ports**: Ensure the following ports are available:
  - `3001-3008`: Microservices
  - `5433`: PostgreSQL
  - `6379`: Redis
  - `5672`, `15672`: RabbitMQ
  - `8080`: API Gateway (if enabled)

---

## Quick Start

### 1. Clone All Repositories

Each service is in a separate repository. Clone them all:

```bash
# Create a workspace directory
mkdir library-system-workspace
cd library-system-workspace

# Clone all service repositories
git clone https://github.com/LibraryBookingSystem/auth-service.git
git clone https://github.com/LibraryBookingSystem/user-service.git
git clone https://github.com/LibraryBookingSystem/booking-service.git
git clone https://github.com/LibraryBookingSystem/catalog-service.git
git clone https://github.com/LibraryBookingSystem/policy-service.git
git clone https://github.com/LibraryBookingSystem/notification-service.git
git clone https://github.com/LibraryBookingSystem/analytics-service.git
git clone https://github.com/LibraryBookingSystem/api-gateway.git
git clone https://github.com/LibraryBookingSystem/docker-compose.git
```

### 2. Navigate to Docker Compose Directory

```bash
cd docker-compose
```

### 3. Start All Services

```bash
docker-compose up -d
```

This will:
- Pull required Docker images
- Build all microservices
- Start infrastructure (PostgreSQL, Redis, RabbitMQ)
- Start all microservices
- Initialize databases

### 4. Verify Services are Running

```bash
docker-compose ps
```

You should see all services with status "Up".

---

## Architecture Overview

The Library System consists of:

### Infrastructure Services
- **PostgreSQL** (port 5433): Main database
- **Redis** (port 6379): Caching layer
- **RabbitMQ** (ports 5672, 15672): Message broker

### Microservices
- **user-service** (port 3001): User management
- **auth-service** (port 3002): Authentication and JWT generation
- **catalog-service** (port 3003): Resource catalog management
- **booking-service** (port 3004): Booking management
- **policy-service** (port 3005): Policy enforcement
- **notification-service** (port 3006): Notification management
- **analytics-service** (port 3007): Analytics and reporting

### API Gateway (Currently Commented Out)
- **api-gateway** (port 8080): Nginx reverse proxy

---

## Running the Application

### Option 1: Docker Compose (Recommended)

This is the easiest way to run the entire system.

#### Start All Services

```bash
cd docker-compose
docker-compose up -d
```

#### Rebuild All Services from Scratch

```bash
cd docker-compose
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

#### Rebuild Specific Service

```bash
cd docker-compose
docker-compose build --no-cache booking-service
docker-compose up -d booking-service
```

#### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f booking-service
docker-compose logs -f auth-service
```

#### Stop All Services

```bash
docker-compose down
```

#### Stop and Remove Volumes (Clears all data)

```bash
docker-compose down -v
```

---

### Option 2: Individual Service Build

If you need to build services one by one (useful for troubleshooting):

```bash
cd docker-compose

# Build infrastructure first
docker-compose up -d postgres redis rabbitmq

# Wait for infrastructure to be healthy, then build services
docker-compose build --no-cache user-service
docker-compose up -d user-service

docker-compose build --no-cache auth-service
docker-compose up -d auth-service

docker-compose build --no-cache catalog-service
docker-compose up -d catalog-service

docker-compose build --no-cache policy-service
docker-compose up -d policy-service

docker-compose build --no-cache booking-service
docker-compose up -d booking-service

docker-compose build --no-cache notification-service
docker-compose up -d notification-service

docker-compose build --no-cache analytics-service
docker-compose up -d analytics-service
```

---

### Option 3: Local Development

For development, you can run services locally without Docker.

#### Prerequisites

1. Install and start PostgreSQL locally
2. Install and start Redis locally
3. Install and start RabbitMQ locally

#### Update Application Configuration

Each service has an `application.yaml` file. Update database and service URLs:

```yaml
# Example: booking-service/src/main/resources/application.yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/booking_db
    username: postgres
    password: postgres
```

#### Run Services

Each service can be run using Maven:

```bash
# In each service directory
cd auth-service
./mvnw spring-boot:run

# Or using Maven directly
mvn spring-boot:run
```

Run services in this order:
1. **user-service** (port 3001)
2. **auth-service** (port 3002)
3. **catalog-service** (port 3003)
4. **policy-service** (port 3005)
5. **booking-service** (port 3004)
6. **notification-service** (port 3006)
7. **analytics-service** (port 3007)

---

## Accessing Services

### Service Endpoints

When running with Docker Compose, services are accessible at:

| Service | Direct Access | Via Gateway (if enabled) |
|---------|--------------|-------------------------|
| User Service | http://localhost:3001 | http://localhost:8080/api/users |
| Auth Service | http://localhost:3002 | http://localhost:8080/api/auth |
| Catalog Service | http://localhost:3003 | http://localhost:8080/api/resources |
| Booking Service | http://localhost:3004 | http://localhost:8080/api/bookings |
| Policy Service | http://localhost:3005 | http://localhost:8080/api/policies |
| Notification Service | http://localhost:3006 | http://localhost:8080/api/notifications |
| Analytics Service | http://localhost:3007 | http://localhost:8080/api/analytics |

### Infrastructure Access

- **PostgreSQL**: `localhost:5433`
  - Username: `postgres`
  - Password: `postgres`
  - Database: `library_system`

- **Redis**: `localhost:6379`

- **RabbitMQ Management UI**: http://localhost:15672
  - Username: `admin`
  - Password: `admin`

### Health Checks

Test if services are running:

```bash
# Auth Service
curl http://localhost:3002/api/auth/health

# User Service
curl http://localhost:3001/api/users/health

# Catalog Service
curl http://localhost:3003/api/resources/health

# Booking Service
curl http://localhost:3004/api/bookings/health

# Policy Service
curl http://localhost:3005/api/policies/health

# Notification Service
curl http://localhost:3006/api/notifications/health

# Analytics Service
curl http://localhost:3007/api/analytics/health
```

---

## Verifying the Setup

### 1. Check Docker Containers

```bash
docker-compose ps
```

All services should show status "Up" and health "healthy" (for infrastructure).

### 2. Check Service Logs

```bash
# Check for errors
docker-compose logs | grep -i error

# Check specific service
docker-compose logs booking-service
```

### 3. Test API Endpoints

#### Register a User

```bash
curl -X POST http://localhost:3002/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "password123",
    "role": "STUDENT"
  }'
```

#### Login

```bash
curl -X POST http://localhost:3002/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "password123"
  }'
```

Save the token from the response for authenticated requests.

#### Get Resources

```bash
curl http://localhost:3003/api/resources \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## Common Operations

### Rebuild After Code Changes

```bash
cd docker-compose

# Rebuild specific service
docker-compose build --no-cache booking-service
docker-compose up -d booking-service

# Rebuild all services
docker-compose build --no-cache
docker-compose up -d
```

### View Service Logs

```bash
# Follow logs for all services
docker-compose logs -f

# Follow logs for specific service
docker-compose logs -f booking-service

# View last 100 lines
docker-compose logs --tail=100 booking-service
```

### Restart a Service

```bash
docker-compose restart booking-service
```

### Stop and Clean Up

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (deletes all data)
docker-compose down -v

# Stop and remove images
docker-compose down --rmi all
```

### Access Service Shell

```bash
# Access PostgreSQL
docker exec -it library-postgres psql -U postgres

# Access Redis CLI
docker exec -it library-redis redis-cli

# Access service container
docker exec -it library-booking-service sh
```

---

## Troubleshooting

### Services Won't Start

1. **Check port availability**:
   ```bash
   # Windows
   netstat -ano | findstr :3001
   
   # Linux/Mac
   lsof -i :3001
   ```

2. **Check Docker resources**:
   - Ensure Docker has enough memory (minimum 4GB)
   - Check Docker Desktop settings

3. **Check logs**:
   ```bash
   docker-compose logs service-name
   ```

### Database Connection Issues

1. **Wait for PostgreSQL to be ready**:
   ```bash
   docker-compose logs postgres | grep "database system is ready"
   ```

2. **Check database initialization**:
   ```bash
   docker exec -it library-postgres psql -U postgres -c "\l"
   ```

3. **Reinitialize databases**:
   ```bash
   docker-compose down -v
   docker-compose up -d postgres
   # Wait for initialization, then start other services
   ```

### Build Failures

1. **Maven dependency issues**:
   - Check network connectivity
   - Some services have Maven settings.xml for better network handling
   - Try rebuilding: `docker-compose build --no-cache service-name`

2. **Out of memory**:
   - Increase Docker memory limit
   - Build services one at a time

### Service Communication Issues

1. **Check Docker network**:
   ```bash
   docker network ls
   docker network inspect docker-compose_library-network
   ```

2. **Verify service names**:
   - Services communicate using Docker service names (e.g., `postgres`, `rabbitmq`)
   - Not `localhost` when running in Docker

3. **Check service health**:
   ```bash
   docker-compose ps
   ```

### Common Error Messages

**"Connection refused"**:
- Service might not be running
- Check with `docker-compose ps`
- Check service logs

**"Port already in use"**:
- Another service is using the port
- Stop conflicting service or change port in docker-compose.yml

**"Health check failed"**:
- Service is starting but not ready
- Wait a few seconds and check again
- Review service logs

---

## Development Workflow

### Making Code Changes

1. **Edit code** in the service directory
2. **Rebuild the service**:
   ```bash
   cd docker-compose
   docker-compose build --no-cache service-name
   docker-compose up -d service-name
   ```
3. **Test the changes**:
   ```bash
   docker-compose logs -f service-name
   ```

### Running Tests

Each service has unit tests. Run them locally:

```bash
cd service-name
./mvnw test
```

### Database Migrations

Database schema is initialized via `init-databases.sql` in the docker-compose directory. To modify:

1. Edit `docker-compose/init-databases.sql`
2. Recreate the database:
   ```bash
   docker-compose down -v
   docker-compose up -d postgres
   ```

### Adding a New Service

1. Create service directory with Dockerfile
2. Add service to `docker-compose.yml`
3. Update API Gateway nginx.conf (if using gateway)
4. Rebuild: `docker-compose build --no-cache new-service`

---

## Environment Variables

Key environment variables (set in docker-compose.yml):

- `JWT_SECRET`: Secret key for JWT token signing
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`: Database connection
- `RABBITMQ_HOST`, `RABBITMQ_PORT`, `RABBITMQ_USER`, `RABBITMQ_PASS`: RabbitMQ connection
- `REDIS_HOST`, `REDIS_PORT`: Redis connection

---

## Next Steps

1. **Read API Documentation**: See [API_DOCUMENTATION.md](API_DOCUMENTATION.md) for complete API reference
2. **Understand RBAC**: See [RBAC_REQUIREMENTS.md](RBAC_REQUIREMENTS.md) for authorization requirements
3. **Test the API**: Use Postman or curl to test endpoints
4. **Explore Services**: Check individual service README files for service-specific details

---

## Support

For issues or questions:
- Check service logs: `docker-compose logs service-name`
- Review this documentation
- Check individual service README files
- Review API documentation for endpoint details

---

**Last Updated**: December 2025
