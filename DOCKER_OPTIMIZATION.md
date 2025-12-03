# Docker Build Optimization Guide

## Overview

This guide explains how to optimize Docker builds to reduce space and network usage by sharing dependencies across services.

## Problem

When building multiple services, each service downloads the same dependencies (Spring Boot, PostgreSQL driver, JWT libraries, etc.) independently, leading to:
- **Wasted network bandwidth**: Same libraries downloaded multiple times
- **Wasted disk space**: Duplicate dependency layers in each image
- **Slower builds**: Each service rebuilds dependencies from scratch

## Solution: Shared Dependency Caching

### Strategy 1: Docker BuildKit Cache Mounts (Recommended)

This uses Docker's BuildKit to share Maven's local repository cache across all builds.

#### Enable BuildKit

```bash
# Windows PowerShell
$env:DOCKER_BUILDKIT=1
$env:COMPOSE_DOCKER_CLI_BUILD=1

# Or set permanently in Docker Desktop settings
```

#### Optimized Dockerfile Template

Create this optimized Dockerfile for each service:

```dockerfile
# syntax=docker/dockerfile:1.4
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app

# Copy Maven settings if available
COPY .mvn/settings.xml* /root/.m2/settings.xml

# Copy pom.xml first (for better layer caching)
COPY pom.xml .

# Download dependencies using shared cache mount (shared across all builds)
RUN --mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared \
    mvn dependency:go-offline -B || mvn dependency:resolve -B

# Copy source code
COPY src ./src

# Build using shared cache mount
RUN --mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared \
    mvn clean package -DskipTests -B

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 3001
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Key Features**:
- `--mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared` - Shares Maven cache across all builds
  - `id=maven-cache` - Names the cache so all services use the same one
  - `sharing=shared` - Allows concurrent access to the cache during parallel builds
- Dependencies are downloaded once and reused across all services
- Only service-specific code changes trigger rebuilds
- **Must build all services together** (using `--parallel`) for cache to work properly

#### Build with BuildKit

**IMPORTANT: Build all services together in parallel for cache sharing**

The cache mount only works effectively when all services are built in the same build session. Building services individually may not share the cache properly.

```bash
# Enable BuildKit
$env:DOCKER_BUILDKIT=1
$env:COMPOSE_DOCKER_CLI_BUILD=1

# Build all services in parallel (RECOMMENDED)
# This ensures all services share the same cache mount during the build session
docker-compose build --parallel

# Or build with verbose output to monitor cache usage
docker-compose build --parallel --progress=plain

# For clean builds (first time or after dependency changes)
docker-compose build --parallel --no-cache
```

**Why parallel builds?**
- All services share the same cache mount during the build session
- Dependencies downloaded by one service are immediately available to others
- Faster overall build time (services build concurrently)
- Maximum cache efficiency

**Building services individually (NOT recommended for cache sharing):**
```bash
# First service - will download dependencies
docker-compose build user-service

# Subsequent services - may not share cache properly
docker-compose build auth-service  # Might still download everything
```

---

### Strategy 2: Shared Base Image with Common Dependencies

Create a base image with all common dependencies pre-downloaded.

#### Step 1: Create Base Image Dockerfile

Create `docker-compose/base-image/Dockerfile`:

```dockerfile
FROM maven:3.9-eclipse-temurin-17

WORKDIR /app

# Create a pom.xml with all common dependencies
COPY base-pom.xml pom.xml

# Download all common dependencies
RUN mvn dependency:go-offline -B

# This layer will be cached and reused
```

#### Step 2: Create Base POM

Create `docker-compose/base-image/base-pom.xml` with all shared dependencies:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.7</version>
    </parent>
    
    <groupId>com.library</groupId>
    <artifactId>base-dependencies</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <!-- Common Spring Boot dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        
        <!-- Common database driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        
        <!-- Common JWT libraries -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.3</version>
        </dependency>
        
        <!-- Common test dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

#### Step 3: Build Base Image

```bash
cd docker-compose/base-image
docker build -t library-base-deps:3.5.7 .
```

#### Step 4: Use Base Image in Service Dockerfiles

```dockerfile
FROM library-base-deps:3.5.7 AS build
WORKDIR /app

# Copy service-specific pom.xml
COPY pom.xml .

# Dependencies are already in base image, just resolve service-specific ones
RUN mvn dependency:resolve -B

# Copy source
COPY src ./src

# Build (will use cached dependencies from base image)
RUN mvn clean package -DskipTests -B

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 3001
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### Strategy 3: Optimized Layer Caching (Current Best Practice)

Improve existing Dockerfiles to maximize layer reuse:

#### Optimized Dockerfile Template

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app

# 1. Copy Maven settings (rarely changes)
COPY .mvn/settings.xml* /root/.m2/settings.xml

# 2. Copy pom.xml first (changes less frequently than source)
COPY pom.xml .

# 3. Download dependencies (cached if pom.xml unchanged)
RUN mvn dependency:go-offline -B || mvn dependency:resolve -B

# 4. Copy source code last (changes most frequently)
COPY src ./src

# 5. Build (only runs if source or pom.xml changed)
RUN mvn clean package -DskipTests -B

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 3001
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Benefits**:
- Dependencies layer is cached separately from source code
- Only rebuilds dependencies when `pom.xml` changes
- Source code changes don't trigger dependency re-download

---

## Implementation Steps

### Option A: Use BuildKit Cache Mounts (Easiest)

1. **Update all Dockerfiles** to use shared cache mounts:

```dockerfile
# Add this line at the top
# syntax=docker/dockerfile:1.4

# In dependency download step (with shared cache):
RUN --mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared \
    mvn dependency:go-offline -B || mvn dependency:resolve -B

# In build step (with shared cache):
RUN --mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared \
    mvn clean package -DskipTests -B
```

**Important**: The `id=maven-cache,sharing=shared` parameters ensure all services share the same cache during parallel builds.

2. **Enable BuildKit**:

```powershell
# PowerShell
$env:DOCKER_BUILDKIT=1
$env:COMPOSE_DOCKER_CLI_BUILD=1

# Or in docker-compose.yml, add at the top:
# x-buildkit: &buildkit
#   DOCKER_BUILDKIT: 1
```

3. **Build all services in parallel** (REQUIRED for cache sharing):

```bash
# Build all services together - this ensures cache sharing
docker-compose build --parallel

# For first-time builds or after dependency changes
docker-compose build --parallel --no-cache

# Monitor cache usage
docker-compose build --parallel --progress=plain
```

**Why parallel builds are essential:**
- The shared cache mount only works effectively when all services are built in the same build session
- Building services individually (`docker-compose build service-name`) may not properly share the cache
- Parallel builds ensure maximum cache efficiency and faster overall build times

### Option B: Create Shared Base Image

1. Create `docker-compose/base-image/` directory
2. Add base Dockerfile and base-pom.xml (see Strategy 2 above)
3. Build base image: `docker build -t library-base-deps:3.5.7 ./base-image`
4. Update service Dockerfiles to use base image
5. Rebuild services

---

## Comparison

| Strategy | Network Savings | Disk Savings | Complexity | Best For |
|----------|----------------|--------------|------------|----------|
| **BuildKit Cache Mounts** | ~80-90% | ~60-70% | Low | All scenarios |
| **Shared Base Image** | ~90-95% | ~70-80% | Medium | CI/CD pipelines |
| **Optimized Layers** | ~50-60% | ~40-50% | Low | Current setup improvement |

---

## Expected Savings

### Before Optimization
- **Network**: ~500MB per service × 7 services = ~3.5GB
- **Disk**: ~800MB per image × 7 services = ~5.6GB
- **Build Time**: ~5-10 minutes per service

### After Optimization (BuildKit)
- **Network**: ~500MB once + ~50MB per service = ~850MB total (75% reduction)
- **Disk**: Shared cache + ~200MB per service = ~2GB total (65% reduction)
- **Build Time**: ~5-10 minutes first build, ~1-2 minutes subsequent builds

---

## Quick Implementation

### Update All Service Dockerfiles

Use this optimized template for all services:

```dockerfile
# syntax=docker/dockerfile:1.4
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app

# Copy Maven settings if available
COPY .mvn/settings.xml* /root/.m2/settings.xml

# Copy pom.xml first
COPY pom.xml .

# Download dependencies with shared cache mount
RUN --mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared \
    mvn dependency:go-offline -B || mvn dependency:resolve -B

# Copy source code
COPY src ./src

# Build with shared cache mount
RUN --mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared \
    mvn clean package -DskipTests -B

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE <PORT>
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Enable BuildKit in docker-compose.yml

Add at the top of `docker-compose.yml`:

```yaml
x-buildkit: &buildkit
  DOCKER_BUILDKIT: 1
  COMPOSE_DOCKER_CLI_BUILD: 1
```

Or set environment variables and build in parallel:

```powershell
# Enable BuildKit
$env:DOCKER_BUILDKIT=1
$env:COMPOSE_DOCKER_CLI_BUILD=1

# Build all services in parallel (ensures cache sharing)
docker-compose build --parallel

# Monitor cache usage during build
docker-compose build --parallel --progress=plain 2>&1 | Select-String -Pattern "Downloading|Using cached|CACHED"
```

**Key Points:**
- Always use `--parallel` flag to build all services together
- This ensures the shared cache mount (`id=maven-cache,sharing=shared`) works across all services
- Building services one-by-one may not properly share the cache between separate build commands

---

## Verification

### Check Cache Usage

```bash
# View BuildKit cache
docker buildx du

# Check image sizes
docker images | grep library-

# Compare before/after
docker system df
```

### Monitor Network Usage

```bash
# Watch network during build
docker-compose build --progress=plain 2>&1 | findstr "Downloading"
```

---

## Additional Optimizations

### 1. Use .dockerignore

Create `.dockerignore` in each service:

```
target/
.mvn/wrapper/maven-wrapper.jar
.git/
.gitignore
*.md
```

### 2. Multi-stage Build Optimization

Already using multi-stage builds (good!). This separates build dependencies from runtime.

### 3. Alpine Base Images

Already using `eclipse-temurin:17-jre-alpine` (good!). Alpine images are ~50% smaller.

### 4. Remove Unnecessary Files

Add to Dockerfile:

```dockerfile
# Remove Maven wrapper and other build artifacts
RUN rm -rf /root/.m2/wrapper
```

---

## Troubleshooting

### BuildKit Not Working

1. **Check Docker version**: Requires Docker 18.09+
2. **Enable BuildKit**: `$env:DOCKER_BUILDKIT=1`
3. **Check syntax**: Must have `# syntax=docker/dockerfile:1.4` at top

### Cache Not Sharing

1. **Verify cache mount syntax**: `--mount=type=cache,target=/root/.m2,id=maven-cache,sharing=shared`
2. **Check BuildKit enabled**: `docker buildx version`
3. **Build all services in parallel**: Use `docker-compose build --parallel` instead of building individually
4. **Verify parallel build**: Check that all services are building concurrently
5. **Check cache mount ID**: All services must use the same `id=maven-cache` for sharing to work

### Base Image Issues

1. **Rebuild base image** when common dependencies change
2. **Tag base image** with version: `library-base-deps:3.5.7`
3. **Update service Dockerfiles** to use correct base image tag

---

## Recommended Approach

**For your setup, I recommend Strategy 1 (BuildKit Cache Mounts)** because:

1. ✅ Easiest to implement (just update Dockerfiles)
2. ✅ No need for separate base image management
3. ✅ Works automatically across all services when built in parallel
4. ✅ Significant savings (75%+ network, 65%+ disk)
5. ✅ Faster subsequent builds (especially with `--parallel` flag)
6. ✅ Shared cache mount ensures dependencies downloaded once and reused across all services

---

## Next Steps

1. Update all service Dockerfiles with BuildKit shared cache mounts (`id=maven-cache,sharing=shared`)
2. Enable BuildKit in your environment (`$env:DOCKER_BUILDKIT=1`)
3. **Build all services in parallel** using `docker-compose build --parallel`
4. Verify cache usage by monitoring "Downloading" vs "Using cached" messages
5. Monitor network and disk usage improvements (expect 75%+ reduction in network usage)

---

**Last Updated**: December 2025
