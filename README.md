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
- [Steps to Initialize Dummy Data](#5-initialize-database-with-dummy-data-optional-but-recommended)
- [Flutter Frontend Application](#flutter-frontend-application)
  - [Flutter App Prerequisites](#flutter-app-prerequisites)
  - [Setting Up the Flutter App](#setting-up-the-flutter-app)
  - [Running the Flutter App with Backend Services](#running-the-flutter-app-with-backend-services)
  - [Using the Flutter App Features](#using-the-flutter-app-features)
  - [Troubleshooting Flutter App](#troubleshooting-flutter-app)
  - [Flutter App Architecture](#flutter-app-architecture)

---

## Prerequisites

Before running the application, ensure you have the following installed:

### Required Software

1. **Docker** (version 20.10 or higher)
   - Download from: <https://www.docker.com/get-started>
   - Verify installation: `docker --version`

2. **Docker Compose** (version 2.0 or higher)
   - Usually included with Docker Desktop
   - Verify installation: `docker-compose --version`

3. **Java 17** (for local development)
   - Download from: <https://adoptium.net/>
   - Verify installation: `java -version`

4. **Maven 3.9+** (for local development)
   - Download from: <https://maven.apache.org/download.cgi>
   - Verify installation: `mvn --version`

5. **Flutter SDK** (for Flutter frontend, version 3.0.0 or higher)
   - Download from: <https://flutter.dev/docs/get-started/install>
   - Verify installation: `flutter --version`
   - Run `flutter doctor` to check for additional requirements

6. **Android Studio / VS Code** (for Flutter development)
   - Android Studio: <https://developer.android.com/studio>
   - VS Code with Flutter extension: <https://code.visualstudio.com/>

### System Requirements

- **RAM**: Minimum 8GB (16GB recommended)
- **Disk Space**: At least 10GB free
- **Ports**: Ensure the following ports are available:
  - `3001-3008`: Microservices
  - `5433`: PostgreSQL
  - `6379`: Redis
  - `5672`, `15672`: RabbitMQ
  - `8080`: API Gateway (enabled by default)

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
git clone https://github.com/LibraryBookingSystem/common-aspects.git
git clone https://github.com/LibraryBookingSystem/docker-compose.git
```

### 2. Navigate to Docker Compose Directory

```bash
cd docker-compose
```

### 3. Complete Setup (Recommended - All-in-One)

**For a complete automated setup**, use the setup script that handles everything:

```powershell
powershell -ExecutionPolicy Bypass -File setup-complete.ps1
```

This script will:

1. ✅ Check if Docker is running
2. ✅ Rebuild all services (with `--no-cache` for clean build)
3. ✅ Start all services
4. ✅ Wait for services to be ready
5. ✅ Verify API Gateway is accessible
6. ✅ Initialize dummy data automatically

**Note**: This script takes several minutes to complete as it rebuilds all services from scratch.

**After the script completes**, you can:
- Access the API Gateway at `http://localhost:8080`
- Log in with admin credentials (username: `admin1`, password: `12345678a`)
- Start using the Flutter frontend application

### 4. Manual Setup (Alternative)

If you prefer manual control or the automated script fails:

#### 4.1. Start All Services

```bash
docker-compose up -d
```

This will:

- Pull required Docker images
- Build all microservices and API Gateway
- Start infrastructure (PostgreSQL, Redis, RabbitMQ)
- Start all microservices
- Start API Gateway (port 8080) - routes all API requests
- Initialize databases

#### 4.2. Verify Services are Running

```bash
docker-compose ps
```

You should see all services with status "Up".

### 5. Initialize Database with Dummy Data (Optional but Recommended)

**Important Notes:**

- **Services Must Be Running**: Ensure all services are running before initializing dummy data. If containers are not running, start them first with `docker-compose up -d`.
- **Spring Boot Auto-Creates Tables**: The Spring Boot services automatically create database tables using JPA/Hibernate when they start.
- **Run After Services Start**: The dummy data script must be run AFTER all services have started and created their tables (wait a few seconds after services show "Up").
- **User Passwords**: User passwords are hashed using BCrypt. For testing, register users through the API or use the actual password hash.

#### Option 1: Automated Script (Recommended)

**Note**: Run from the `docker-compose` directory.

Use the automated PowerShell script that handles everything:

```powershell
powershell -ExecutionPolicy Bypass -File init-dummy-data-all.ps1
```

This script will:

1. Copy SQL files to the container
2. Insert catalog data (resources) into `catalog_db`
3. Insert policy data (booking policies) into `policy_db`
4. Create and approve admin user via API

#### Option 2: Using psql

1. **Ensure services are running**:

   ```bash
   cd docker-compose
   docker-compose ps
   ```

   If no containers are running, start them first:

   ```bash
   docker-compose up -d
   ```

2. **Wait for all services to start and create tables**:

   ```bash
   docker-compose ps
   ```

   Ensure all services show "Up" status, then wait a few seconds for Spring Boot to create tables.

3. **Connect to PostgreSQL**:

   ```bash
   docker exec -it library-postgres psql -U postgres
   ```

4. **Run the dummy data script**:

   ```sql
   \i /docker-entrypoint-initdb.d/init-dummy-data.sql
   ```

   Or copy and paste the contents of `init-dummy-data.sql` directly (when in the `docker-compose` directory).

#### Option 3: Manual Execution Per Database

**Note**: Run these commands from the `docker-compose` directory.

1. **Copy the scripts into the container**:

   ```powershell
   docker cp init-dummy-data-catalog.sql library-postgres:/tmp/init-dummy-data-catalog.sql
   docker cp init-dummy-data-policy.sql library-postgres:/tmp/init-dummy-data-policy.sql
   ```

2. **Execute the scripts** (wait 10-30 seconds after services start for tables to be created):

   ```powershell
   # Insert catalog data (resources)
   docker exec -i library-postgres psql -U postgres -d catalog_db -f /tmp/init-dummy-data-catalog.sql
   
   # Insert policy data
   docker exec -i library-postgres psql -U postgres -d policy_db -f /tmp/init-dummy-data-policy.sql
   
   # Create admin user (via API)
   powershell -ExecutionPolicy Bypass -File setup-admin-user.ps1
   ```

#### Option 4: Using Merged File

**Note**: The merged `init-dummy-data.sql` contains all sections but targets different databases. Extract and run each section separately.

1. **Copy the merged script into the container**:

   ```powershell
   # From docker-compose directory
   docker cp init-dummy-data.sql library-postgres:/tmp/init-dummy-data.sql
   
   # Or from workspace root
   docker cp docker-compose/init-dummy-data.sql library-postgres:/tmp/init-dummy-data.sql
   ```

2. **Extract and run each section** (since each targets a different database):
   - Section 1: Copy catalog SQL and run in `catalog_db`
   - Section 2: Copy policy SQL and run in `policy_db`
   - Section 3: Use `setup-admin-user.ps1` for admin user

#### Option 5: Manual Execution

1. **Connect to PostgreSQL**:

   ```bash
   docker exec -it library-postgres psql -U postgres
   ```

2. **Switch to each database and run the appropriate INSERT statements manually** from `init-dummy-data.sql` (located in the `docker-compose` directory).

#### What Gets Created

- **Resources**: 18 total (6 study rooms, 6 computer stations, 6 seats) + amenities
- **Policies**: 4 default booking policies (Student, Faculty, Admin, Peak Hours)
- **Admin User**: 1 hardcoded admin (username: `admin1`, password: `12345678a`) - created via API
- **Bookings**: Created dynamically by users through the application
- **Notifications**: Created dynamically by the system

#### File Structure

- **`init-dummy-data.sql`** - **Merged file** containing all sections (catalog, policy, user)
- **`init-dummy-data-catalog.sql`** - Section 1 extract for `catalog_db` (convenience)
- **`init-dummy-data-policy.sql`** - Section 2 extract for `policy_db` (convenience)
- **`setup-admin-user.ps1`** - **Unified admin user script** (replaces multiple admin scripts)
- **`init-dummy-data-all.ps1`** - Automated script to run all initialization

#### Admin User Management

The `setup-admin-user.ps1` script is a unified tool for admin user operations:

```powershell
# Full setup (create new or approve existing)
.\setup-admin-user.ps1

# Only approve existing user
.\setup-admin-user.ps1 -ApproveOnly

# Force recreate (delete and create new)
.\setup-admin-user.ps1 -Recreate
```

#### Verifying Data

After running the script, verify data was inserted:

```bash
docker exec -it library-postgres psql -U postgres -d catalog_db -c "SELECT COUNT(*) FROM resources;"
docker exec -it library-postgres psql -U postgres -d policy_db -c "SELECT COUNT(*) FROM booking_policies;"
```

**Troubleshooting:**

- **"No such container"**: Services are not running. Start them with `docker-compose up -d` from the `docker-compose` directory.
- **"relation does not exist"**: Services haven't created tables yet. Wait a few more seconds and try again.
- **"duplicate key"**: Data already exists. Drop and recreate tables or skip duplicate inserts.
- **Connection refused**: PostgreSQL container isn't running. Start it with `docker-compose up -d postgres`.
- **"The system cannot find the file specified"**: Ensure you're using the correct path. From `docker-compose` directory, use `init-dummy-data.sql`. From workspace root, use `docker-compose/init-dummy-data.sql`.

### 6. Clone Flutter Frontend (Optional but Recommended)

The Flutter frontend provides a complete user interface for the library booking system:

```bash
# If not already cloned
cd library-system-workspace
git clone https://github.com/LibraryBookingSystem/frontend-web.git
```

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

### API Gateway ✅ Enabled

- **api-gateway** (port 8080): Nginx reverse proxy
  - Routes all API requests to appropriate microservices
  - Handles CORS for Flutter web app
  - Single entry point for all backend services
  - Configured and ready to use

### Frontend Application

- **Flutter Frontend** (`frontend-web`): Mobile and web application
  - Connects to backend services via API Gateway (`http://localhost:8080`)
  - Uses WebSocket for real-time updates
  - Implements AOP (Aspect-Oriented Programming) and SOA (Service-Oriented Architecture)

### API Gateway Configuration

The API Gateway is **enabled by default** and provides:

- **Single Entry Point**: All API requests go through `http://localhost:8080`
- **Request Routing**: Automatically routes requests to appropriate microservices
- **CORS Support**: Configured for Flutter web app cross-origin requests
- **Health Monitoring**: `/health` endpoint for gateway status

**Configuration Files**:

- `api-gateway/Dockerfile` - Nginx-based gateway container
- `api-gateway/nginx.conf` - Routing and CORS configuration
- `docker-compose.yml` - Gateway service definition

**Routes**:

- `/api/auth/*` → auth-service:3002
- `/api/users/*` → user-service:3001
- `/api/resources/*` → catalog-service:3003
- `/api/bookings/*` → booking-service:3004
- `/api/policies/*` → policy-service:3005
- `/api/notifications/*` → notification-service:3006
- `/api/analytics/*` → analytics-service:3007

---

## Running the Application

### Option 1: Docker Compose (Recommended)

This is the easiest way to run the entire system.

#### Start All Services (Including API Gateway)

```bash
cd docker-compose
docker-compose up -d
```

This starts all services including the **API Gateway** on port 8080. The API Gateway provides a single entry point for all API requests and handles CORS for web applications.

#### Rebuild All Services from Scratch

```bash
cd docker-compose
docker-compose down
docker-compose build --parallel --no-cache
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

| Service | Direct Access | Via Gateway (Recommended) |
|---------|--------------|-------------------------|
| User Service | <http://localhost:3001> | <http://localhost:8080/api/users> |
| Auth Service | <http://localhost:3002> | <http://localhost:8080/api/auth> |
| Catalog Service | <http://localhost:3003> | <http://localhost:8080/api/resources> |
| Booking Service | <http://localhost:3004> | <http://localhost:8080/api/bookings> |
| Policy Service | <http://localhost:3005> | <http://localhost:8080/api/policies> |
| Notification Service | <http://localhost:3006> | <http://localhost:8080/api/notifications> |
| Analytics Service | <http://localhost:3007> | <http://localhost:8080/api/analytics> |

**Note**: The API Gateway is now enabled by default. All Flutter app requests should go through `http://localhost:8080`.

### Infrastructure Access

- **PostgreSQL**: `localhost:5433`
  - Username: `postgres`
  - Password: `postgres`
  - Database: `library_system`

- **Redis**: `localhost:6379`

- **RabbitMQ Management UI**: <http://localhost:15672>
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

### 3. Verify Database Initialization

Check that databases were created and tables exist:

```bash
# List all databases
docker exec -it library-postgres psql -U postgres -c "\l"

# Check if tables were created (example for catalog_db)
docker exec -it library-postgres psql -U postgres -d catalog_db -c "\dt"

# Verify dummy data (if initialized)
docker exec -it library-postgres psql -U postgres -d catalog_db -c "SELECT COUNT(*) FROM resources;"
docker exec -it library-postgres psql -U postgres -d policy_db -c "SELECT COUNT(*) FROM booking_policies;"
```

### 4. Test API Endpoints

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
docker-compose build --parallel --no-cache
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

3. **Wait for Spring Boot to create tables**:
   - After services start, wait a few seconds for JPA/Hibernate to create tables
   - Check if tables exist: `docker exec -it library-postgres psql -U postgres -d catalog_db -c "\dt"`

4. **Reinitialize databases**:

   ```bash
   docker-compose down -v
   docker-compose up -d postgres
   # Wait for initialization, then start other services
   # After services start, wait for tables to be created before running dummy data script
   ```

### Database Dummy Data Issues

1. **"relation does not exist" error**:
   - Services haven't created tables yet
   - Wait a few more seconds after services show "Up" status
   - Verify tables exist: `docker exec -it library-postgres psql -U postgres -d catalog_db -c "\dt"`

2. **"duplicate key" error**:
   - Data already exists in the database
   - Scripts use `ON CONFLICT DO NOTHING` and `IF NOT EXISTS` to prevent duplicates
   - Safe to re-run scripts
   - To start fresh: `docker-compose down -v` then restart services

3. **Script not found**:
   - Ensure you're in the correct directory: `cd docker-compose`
   - Verify file exists: `ls init-dummy-data.sql`
   - Use the automated script: `init-dummy-data-all.ps1` (recommended)
   - Or use separate convenience files: `init-dummy-data-catalog.sql`, `init-dummy-data-policy.sql`

4. **Admin user login issues**:
   - Use `setup-admin-user.ps1` to create/approve admin user (unified script)
   - Check if user exists: `docker exec library-postgres psql -U postgres -d user_db -c "SELECT * FROM users WHERE username = 'admin1';"`
   - Verify user is approved: `pending_approval` should be `false`
   - Admin credentials: username `admin1`, password `12345678a`

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

**Note**:

- Spring Boot services automatically create tables using JPA/Hibernate when they start
- The `init-databases.sql` script creates the database schemas
- The `init-dummy-data.sql` script (optional) populates initial test data
- After recreating databases, wait for services to create tables before running dummy data script

### Adding a New Service

1. Create service directory with Dockerfile
2. Add service to `docker-compose.yml`
3. Update API Gateway `nginx.conf` to add routing for the new service:

   ```nginx
   location /api/new-service {
       proxy_pass http://new-service:PORT;
       # ... proxy settings
   }
   ```

4. Rebuild: `docker-compose build --no-cache new-service`
5. Restart API Gateway: `docker-compose restart api-gateway`

---

## Environment Variables

Key environment variables (set in docker-compose.yml):

- `JWT_SECRET`: Secret key for JWT token signing
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`: Database connection
- `RABBITMQ_HOST`, `RABBITMQ_PORT`, `RABBITMQ_USER`, `RABBITMQ_PASS`: RabbitMQ connection
- `REDIS_HOST`, `REDIS_PORT`: Redis connection

---

## Flutter Frontend Application

The Flutter frontend provides a complete user interface for the library booking system, built with **Aspect-Oriented Programming (AOP)** and **Service-Oriented Architecture (SOA)** principles.

### Flutter App Prerequisites

1. **Flutter SDK** (>=3.0.0)
   - Download from: <https://flutter.dev/docs/get-started/install>
   - Verify: `flutter --version`
   - Run `flutter doctor` to check setup

2. **Android Studio** (for Android development)
   - Android SDK
   - Android emulator or physical device

3. **Xcode** (for iOS development, macOS only)
   - Required for iOS builds
   - iOS Simulator or physical device

4. **Backend Services Running**
   - All backend services must be running (see [Running the Application](#running-the-application))
   - Services should be accessible at `http://localhost:3001-3007`

### Setting Up the Flutter App

#### 1. Navigate to Flutter App Directory

```bash
cd library-system-workspace/frontend-web
```

#### 2. Install Dependencies

```bash
flutter pub get
```

#### 3. Configure API Endpoints

The app is pre-configured to connect to the API Gateway at `http://localhost:8080`. The API Gateway routes all requests to the appropriate microservices.

**File**: `lib/core/config/app_config.dart`

```dart
class AppConfig {
  // API Gateway (recommended - routes to all services)
  static const String baseApiUrl = 'http://localhost:8080';
  static const String websocketUrl = 'ws://localhost:8080/ws/availability';
}
```

**Note**:

- ✅ **Default**: The app is configured to use the API Gateway at `http://localhost:8080` (recommended)
- The API Gateway is enabled by default in docker-compose
- For Android emulator, change to `http://10.0.2.2:8080`
- For physical Android device, use your computer's IP address (e.g., `http://192.168.1.100:8080`)

#### 4. Initialize Platform-Specific Code

The app is already initialized for Android and iOS. If needed:

```bash
# For Android
flutter create --platforms=android .

# For iOS (macOS only)
flutter create --platforms=ios .
```

### Running the Flutter App with Backend Services

#### Step 1: Start Backend Services

First, ensure all backend services are running:

```bash
cd library-system-workspace/docker-compose
docker-compose up -d
```

Wait for all services to be healthy:

```bash
docker-compose ps
```

Verify services are responding:

```bash
# Check API Gateway (recommended - routes to all services)
curl http://localhost:8080/health
curl http://localhost:8080/api/auth/health

# Or check services directly
curl http://localhost:3002/api/auth/health
curl http://localhost:3001/api/users/health
curl http://localhost:3003/api/resources/health
```

#### Step 2: Run Flutter App

**For Android:**

```bash
cd library-system-workspace/frontend-web

# List available devices
flutter devices

# Run on connected device/emulator
flutter run

# Or build APK
flutter build apk
```

**For iOS (macOS only):**

```bash
cd library-system-workspace/frontend-web

# Run on iOS Simulator
flutter run

# Or build for iOS
flutter build ios
```

**For Web:**

```bash
cd library-system-workspace/frontend-web
flutter run -d chrome
```

#### Step 3: Verify Connection

When the app starts:

1. You should see the login/register screen
2. The app will attempt to connect to backend services
3. Check Flutter console for any connection errors

### Using the Flutter App Features

#### User Registration and Login

1. **Register a New Account**:
   - Open the app
   - Tap "Register" or "Sign Up"
   - Fill in:
     - Username
     - Email
     - Password
     - Confirm Password
     - Role (Student, Staff, or Admin)
   - Tap "Register"
   - You'll be automatically logged in

2. **Login**:
   - Enter username and password
   - Tap "Login"
   - You'll be redirected to your role-specific dashboard

#### Student Features

**Home Dashboard**:

- View quick statistics
- See upcoming bookings
- Quick access to main features

**Browse Resources**:

- Tap "Browse Resources" from home
- View all available library resources (rooms, equipment, books)
- Use filters:
  - Resource type (Room, Equipment, Book)
  - Floor number
  - Status (Available, Occupied, Maintenance)
- Search by name or description
- Tap a resource to see details

**Interactive Floor Plan**:

- Tap "Floor Plan" from home
- View interactive SVG-based floor plan
- See real-time availability (color-coded)
- Tap resources on the map to view details or book

**Create Booking**:

- Tap "Create Booking" or select a resource
- Choose:
  - Resource (if not pre-selected)
  - Start date and time
  - End date and time
- The app validates against booking policies
- Tap "Create Booking"
- On success, a QR code is generated for check-in

**My Bookings**:

- Tap "My Bookings" from home
- View all your bookings:
  - **Upcoming**: Future bookings
  - **Active**: Current bookings
  - **Past**: Completed bookings
  - **Cancelled**: Cancelled bookings
- Tap a booking to see details
- Cancel upcoming bookings (if allowed)

**Booking Details**:

- View full booking information
- See QR code for check-in
- View booking status
- Cancel booking (if allowed)

**Check-In**:

- Tap "Check-In" from home or booking details
- Two options:
  - **Scan QR Code**: Use device camera to scan booking QR code
  - **Manual Entry**: Enter QR code manually
- On successful check-in, booking status updates

**Notifications**:

- View real-time notifications
- See unread notification count
- Mark notifications as read

#### Staff Features

Staff have access to **all Student features**, plus:

**Staff Dashboard**:

- View staff-specific quick actions
- Access to student features
- Staff-only features

**Occupancy Overview**:

- Tap "Occupancy Overview"
- View real-time statistics:
  - Current occupancy
  - Resource utilization
  - Peak hours
- See charts and graphs

**Manual Check-In**:

- Tap "Manual Check-In"
- Search for user by username
- Select user's booking
- Manually check in the booking
- Useful for assisting students

#### Admin Features

Admins have access to **all Staff features**, plus:

**Admin Dashboard**:

- View key metrics
- Quick access to admin features
- System overview

**Resource Management**:

- Tap "Resource Management"
- View all resources
- **Create Resource**:
  - Tap "+" or "Add Resource"
  - Fill in:
    - Name
    - Description
    - Type (Room, Equipment, Book)
    - Floor
    - Capacity
    - Status
  - Tap "Create"
- **Edit Resource**:
  - Tap a resource
  - Tap "Edit"
  - Update fields
  - Tap "Save"
- **Delete Resource**:
  - Tap a resource
  - Tap "Delete"
  - Confirm deletion

**Policy Configuration**:

- Tap "Policy Configuration"
- View all booking policies
- **Create Policy**:
  - Tap "+" or "Add Policy"
  - Fill in:
    - Name
    - Description
    - Rule Type (Max Duration, Advance Booking, etc.)
    - Rule Value
    - Active status
  - Tap "Create"
- **Edit Policy**:
  - Tap a policy
  - Tap "Edit"
  - Update fields
  - Tap "Save"
- **Delete Policy**:
  - Tap a policy
  - Tap "Delete"
  - Confirm deletion

**User Management**:

- Tap "User Management"
- View all users
- Search users by username or email
- **Restrict User**:
  - Tap a user
  - Tap "Restrict"
  - Enter restriction reason
  - Confirm
- **Unrestrict User**:
  - Tap a restricted user
  - Tap "Unrestrict"
  - Confirm

**Analytics**:

- Tap "Analytics"
- Select date range
- View:
  - **Utilization Statistics**: Resource usage over time
  - **Peak Hours**: Busiest times
  - **Overall Stats**: System-wide statistics
- Charts and graphs visualize the data

**Audit Logs** (when API is available):

- Tap "Audit Logs"
- View system audit logs
- Filter by date, user, or action

### Real-time Features

The app supports real-time updates:

1. **Resource Availability**:
   - Availability updates automatically via WebSocket
   - Falls back to polling if WebSocket unavailable
   - See real-time status changes on floor plan and resource lists

2. **Notifications**:
   - Receive real-time notifications
   - Unread count updates automatically
   - Push notifications for important events

### Troubleshooting Flutter App

#### App Won't Connect to Backend

1. **Verify Backend Services and API Gateway**:

   ```bash
   docker-compose ps
   # Check API Gateway (recommended)
   curl http://localhost:8080/health
   curl http://localhost:8080/api/auth/health
   # Or check services directly
   curl http://localhost:3002/api/auth/health
   ```

2. **Check API Configuration**:
   - Verify `lib/core/config/app_config.dart` has `baseApiUrl = 'http://localhost:8080'` (API Gateway)
   - Ensure API Gateway is running: `docker-compose ps api-gateway`
   - For Android emulator, use `http://10.0.2.2:8080`

3. **Check Network**:
   - Ensure device/emulator can reach `localhost`
   - For Android emulator, use `10.0.2.2` instead of `localhost`
   - For iOS Simulator, `localhost` works

4. **Update API Config for Emulator**:

   ```dart
   // For Android emulator
   static const String baseUrl = 'http://10.0.2.2:8080';
   
   // For physical device (use your computer's IP)
   static const String baseUrl = 'http://192.168.1.100:8080';
   ```

#### Build Errors

1. **Clean and Rebuild**:

   ```bash
   flutter clean
   flutter pub get
   flutter build apk  # or flutter run
   ```

2. **Check Flutter Version**:

   ```bash
   flutter --version  # Should be >=3.0.0
   flutter doctor
   ```

3. **Platform-Specific Issues**:
   - **Android**: Check `android/app/build.gradle` for SDK versions
   - **iOS**: Check `ios/Podfile` and run `pod install` in `ios/` directory

#### QR Code Scanner Not Working

1. **Check Permissions**:
   - Android: Ensure camera permission in `AndroidManifest.xml`
   - iOS: Ensure camera permission in `Info.plist`
   - Grant permission when app requests it

2. **Use Manual Entry**:
   - If scanner fails, use manual QR code entry option

#### WebSocket Connection Issues

- App automatically falls back to polling
- Check WebSocket URL in `app_config.dart`
- Verify WebSocket server is running (if using realtime-gateway)

#### API Gateway Connection Issues

1. **Verify API Gateway is Running**:

   ```bash
   docker-compose ps api-gateway
   # Should show "Up" status
   ```

2. **Check Gateway Health**:

   ```bash
   curl http://localhost:8080/health
   # Should return "API Gateway is running!"
   ```

3. **Check Gateway Logs**:

   ```bash
   docker-compose logs api-gateway
   # Look for nginx configuration errors
   ```

4. **Restart API Gateway**:

   ```bash
   docker-compose restart api-gateway
   ```

5. **Rebuild API Gateway** (if configuration changed):

   ```bash
   docker-compose build api-gateway
   docker-compose up -d api-gateway
   ```

### Flutter App Architecture

The Flutter app follows:

- **Aspect-Oriented Programming (AOP)**: Cross-cutting concerns (logging, auth, error handling) via interceptors and mixins
- **Service-Oriented Architecture (SOA)**: Independent services for each backend microservice
- **Provider State Management**: Reactive state management
- **Real-time Updates**: WebSocket with polling fallback

For detailed architecture documentation, see `frontend-web/README.md` and `frontend-web/AOP_SOA_ARCHITECTURE.md`.

---

## Next Steps

1. **Read API Documentation**: See [API_DOCUMENTATION.md](API_DOCUMENTATION.md) for complete API reference
2. **Understand RBAC**: See [RBAC_REQUIREMENTS.md](RBAC_REQUIREMENTS.md) for authorization requirements
3. **Test the API**: Use Postman or curl to test endpoints
4. **Explore Services**: Check individual service README files for service-specific details
5. **Set Up Flutter Frontend**: Follow the [Flutter Frontend Application](#flutter-frontend-application) section to run the mobile/web app
6. **Read Flutter Documentation**: See `frontend-web/README.md` for detailed Flutter app architecture and features

---

## Support

For issues or questions:

- Check service logs: `docker-compose logs service-name`
- Review this documentation
- Check individual service README files
- Review API documentation for endpoint details

---

**Last Updated**: December 2025
