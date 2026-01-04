# Vartur Services Overview

All services are running on the **`vartur`** Docker network, allowing them to communicate with each other using container names.

## Infrastructure Services

| Service | Container Name | Port | Internal URL | External URL |
|---------|---------------|------|--------------|--------------|
| MySQL | `mysql` | 3306 | `mysql:3306` | `localhost:3306` |
| MongoDB | `mongo` | 27017 | `mongo:27017` | `localhost:27017` |
| Redis | `redis` | 6379 | `redis:6379` | `localhost:6379` |
| RabbitMQ | `rabbitmq` | 5672, 15672 | `rabbitmq:5672` | `localhost:5672` |
| Elasticsearch | `elasticsearch` | 9200, 9300 | `elasticsearch:9200` | `localhost:9200` |
| Kibana | `kibana` | 5601 | `kibana:5601` | `localhost:5601` |

## Application Services

### Backend APIs

| Service | Container Name | Port | Internal URL | External URL | Dockerfile |
|---------|---------------|------|--------------|--------------|------------|
| Public API | `vcrm-api` | 3009 | `http://vcrm-api:3009` | `http://localhost:3009` | `vcrm-api.Dockerfile` |
| Account API | `vcrm-account-api` | 5555 | `http://vcrm-account-api:5555` | `http://localhost:5555` | `vcrm-account-api.Dockerfile` |
| Admin API | `vcrm-admin-api` | 1111 | `http://vcrm-admin-api:1111` | `http://localhost:1111` | `vcrm-admin-api.Dockerfile` |

### Frontend Applications

| Service | Container Name | Port | Internal URL | External URL | Dockerfile |
|---------|---------------|------|--------------|--------------|------------|
| Vartur Desktop | `vartur-desktop` | 3366 | `http://vartur-desktop:3366` | `http://localhost:3366` | `vartur-desktop.Dockerfile` |
| Ilan TR Desktop | `ilan-tr-desktop` | 6644 | `http://ilan-tr-desktop:6644` | `http://localhost:6644` | `ilan-web.Dockerfile` |

### Background Services (NestJS)

| Service | Container Name | Dockerfile | Purpose |
|---------|---------------|------------|---------|
| Cache Service | `vcrm-cache` | `vcrm-cache.Dockerfile` | Handles caching operations |
| Notification Service | `vcrm-notification` | `vcrm-notification.Dockerfile` | Sends notifications |
| Import Service | `vcrm-import` | `vcrm-import.Dockerfile` | Data import operations |
| Customer Service | `vcrm-customers` | `vcrm-customers.Dockerfile` | Customer management |

### Queue Consumers

| Service | Container Name | Dockerfile | Queue |
|---------|---------------|------------|-------|
| Account Queue | `vcrm-queues-account` | `vcrm-queues.Dockerfile` | `account_queue` |
| Admin Queue | `vcrm-queues-admin` | `vcrm-queues.Dockerfile` | `admin_queue` |
| API Queue | `vcrm-queues-api` | `vcrm-queues.Dockerfile` | `api_queue` |

### Utility Services

| Service | Container Name | Dockerfile | Purpose |
|---------|---------------|------------|---------|
| Clean Cache | `vcrm-clean-cache` | `vcrm-clean-cache.Dockerfile` | Cache cleanup |
| Migration | `vcrm-migration` | `vcrm-migration.Dockerfile` | Database migrations (profile: migration) |

## Network Communication

### Frontend to Backend

The frontend applications use **dual URL configuration**:

#### Server-Side (SSR) - Internal Network
```env
API_BASE=http://vcrm-api:3009/v1
VCRM_ACCOUNT_API_URL=http://vcrm-account-api:5555/v1
VCRM_ACCOUNT_SOCKET_URL=http://vcrm-account-api:5555
```

#### Client-Side (Browser) - External Access
```env
NUXT_PUBLIC_API_BASE=http://localhost:3009/v1
NUXT_PUBLIC_VCRM_ACCOUNT_API_URL=http://localhost:5555/v1
NUXT_PUBLIC_VCRM_ACCOUNT_SOCKET_URL=http://localhost:5555
```

### Backend to Backend

All backend services communicate using container names:
- `vcrm-api` → `redis:6379`
- `vcrm-account-api` → `mysql:3306`, `mongo:27017`, `rabbitmq:5672`
- `vcrm-admin-api` → `mysql:3306`, `mongo:27017`, `rabbitmq:5672`
- All services → `elasticsearch:9200`

### Service Dependencies

```
vartur-desktop
  ├── vcrm-api (healthy)
  ├── vcrm-account-api (healthy)
  └── rabbitmq (healthy)

vcrm-api
  ├── redis (healthy)
  ├── elasticsearch (healthy)
  ├── mongo (healthy)
  └── rabbitmq (healthy)

vcrm-account-api
  ├── mysql (healthy)
  ├── mongo (healthy)
  ├── rabbitmq (healthy)
  ├── redis (healthy)
  └── elasticsearch (healthy)

vcrm-admin-api
  ├── mysql (healthy)
  ├── mongo (healthy)
  ├── rabbitmq (healthy)
  ├── redis (healthy)
  └── elasticsearch (healthy)
```

## Environment Files

All services use environment files located in `docker/env/`:

- `vcrm-api.env` - Public API configuration
- `vcrm-account-api.env` - Account API configuration
- `vcrm-admin-api.env` - Admin API configuration
- `vartur-desktop.env` - Frontend configuration
- `vcrm-cache.env` - Cache service configuration
- `vcrm-notification.env` - Notification service configuration
- `vcrm-import.env` - Import service configuration
- `vcrm-customers.env` - Customer service configuration

## Quick Start

### Start All Services
```bash
cd docker
docker compose up -d
```

### Start Infrastructure Only
```bash
docker compose up -d mysql mongo redis rabbitmq elasticsearch kibana
```

### Start Backend APIs
```bash
docker compose up -d vcrm-api vcrm-account-api vcrm-admin-api
```

### Start Frontend
```bash
docker compose up -d vartur-desktop
```

### Run Migrations
```bash
docker compose --profile migration up vcrm-migration
```

### Check Service Health
```bash
docker compose ps
```

### View Logs
```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f vcrm-api
docker compose logs -f vartur-desktop
```

## Access Points

After starting all services, access them at:

| Service | URL |
|---------|-----|
| Vartur Desktop (Frontend) | http://localhost:3366 |
| Ilan TR Desktop (Frontend) | http://localhost:6644 |
| Public API | http://localhost:3009 |
| Public API Docs | http://localhost:3009/docs |
| Account API | http://localhost:5555 |
| Account API Docs | http://localhost:5555/docs |
| Admin API | http://localhost:1111 |
| Admin API Docs | http://localhost:1111/docs |
| RabbitMQ Management | http://localhost:15672 |
| Kibana | http://localhost:5601 |
| Elasticsearch | http://localhost:9200 |

## Health Checks

All application services have health checks configured:

- **Backend APIs**: Check `/docs` endpoint
- **Frontend**: Check root `/` endpoint
- **Infrastructure**: Native health check commands

Health checks include:
- 30-40s start period (60s for frontend)
- 30s interval between checks
- 10s timeout
- 3 retries before marking unhealthy

## Troubleshooting

### Service Not Starting
```bash
# Check logs
docker compose logs -f <service-name>

# Check health status
docker compose ps
```

### Frontend Can't Connect to Backend
Verify environment variables in `docker/env/vartur-desktop.env`:
- Internal URLs use container names (e.g., `vcrm-api:3009`)
- Public URLs use localhost (e.g., `localhost:3009`)

### Database Connection Issues
```bash
# Restart infrastructure
docker compose restart mysql mongo redis

# Check if databases are healthy
docker compose ps mysql mongo redis
```

### Port Conflicts
```bash
# Check what's using a port
lsof -i :3009

# Stop conflicting service or change port in docker-compose.yml
```
