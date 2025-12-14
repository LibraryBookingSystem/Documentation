# Library System Documentation

## Quick Start

```bash
cd docker-compose
docker-compose up -d
```

Services available at `http://localhost:8080` (API Gateway)

## Default Credentials

| User   | Password  | Role  |
| ------ | --------- | ----- |
| admin1 | 12345678a | ADMIN |

## Documentation Index

| File                                                       | Description               |
| ---------------------------------------------------------- | ------------------------- |
| [API_REFERENCE.md](API_REFERENCE.md)                       | All API endpoints         |
| [AUTHORIZATION.md](AUTHORIZATION.md)                       | RBAC & JWT authentication |
| [../POSTMAN_TESTING_GUIDE.md](../POSTMAN_TESTING_GUIDE.md) | Testing guide             |

## Architecture

```
┌─────────────────┐
│   API Gateway   │ :8080
│     (nginx)     │
└────────┬────────┘
         │
    ┌────┴────┬────────┬────────┬────────┬────────┬────────┐
    ▼         ▼        ▼        ▼        ▼        ▼        ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│ Auth  │ │ User  │ │Catalog│ │Booking│ │Policy │ │Notif. │ │Analyt.│
│ :3002 │ │ :3001 │ │ :3003 │ │ :3004 │ │ :3005 │ │ :3006 │ │ :3007 │
└───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘
    │         │        │        │        │        │        │
    └─────────┴────────┴────────┴────────┴────────┴────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         ┌─────────┐    ┌─────────┐    ┌─────────┐
         │PostgreSQL│    │  Redis  │    │RabbitMQ │
         │  :5433  │    │  :6379  │    │  :5672  │
         └─────────┘    └─────────┘    └─────────┘
```

## Service Ports

| Service              | Port | Endpoint             |
| -------------------- | ---- | -------------------- |
| API Gateway          | 8080 | `/api/*`             |
| User Service         | 3001 | `/api/users`         |
| Auth Service         | 3002 | `/api/auth`          |
| Catalog Service      | 3003 | `/api/resources`     |
| Booking Service      | 3004 | `/api/bookings`      |
| Policy Service       | 3005 | `/api/policies`      |
| Notification Service | 3006 | `/api/notifications` |
| Analytics Service    | 3007 | `/api/analytics`     |

## Common Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service-name]

# Rebuild service
docker-compose build --no-cache [service-name]
docker-compose up -d [service-name]

# Stop all
docker-compose down

# Reset (delete data)
docker-compose down -v
```

## Health Checks

```bash
curl http://localhost:8080/health
curl http://localhost:8080/api/auth/health
curl http://localhost:8080/api/users/health
```

---

**Last Updated**: December 2025
