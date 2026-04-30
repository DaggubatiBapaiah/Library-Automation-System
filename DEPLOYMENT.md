# Enterprise Deployment Guide

## 🚀 Quick Start with Docker

```bash
# 1. Clone repository
git clone <repository-url>
cd freelancing-1

# 2. Create .env file
cp .env.example .env

# 3. Update .env with production values
nano .env

# 4. Start all services
docker-compose up --build
```

Services will be available at:
- **API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Prometheus**: http://localhost:9090 (with --profile monitoring)
- **Grafana**: http://localhost:3001 (with --profile monitoring)

---

## 📦 Services Architecture

### 1. PostgreSQL Database
- **Container**: `library_postgres`
- **Port**: 5432
- **Persistent Volume**: `postgres_data`
- **Health Check**: Automated pg_isready

### 2. Redis Cache
- **Container**: `library_redis`
- **Port**: 6379
- **Purpose**: WebSocket scaling, rate limiting, caching
- **Persistent Volume**: `redis_data`

### 3. FastAPI Application
- **Container**: `library_api`
- **Port**: 8000
- **Workers**: 4 (Gunicorn + UvicornWorker)
- **Features**:
  - Multi-worker WebSocket scaling via Redis pub/sub
  - Prometheus metrics endpoint
  - Health checks
  - Auto-restart on failure

---

## 🗄️ Database Migrations with Alembic

### Initial Setup
```bash
# Generate initial migration
alembic revision --autogenerate -m "Initial migration"

# Apply migrations
alembic upgrade head
```

### Creating New Migrations
```bash
# After modifying models
alembic revision --autogenerate -m "Description of changes"

# Review generated migration in alembic/versions/
alembic upgrade head
```

### Rollback
```bash
# Downgrade one version
alembic downgrade -1

# Downgrade to specific version
alembic downgrade <revision_id>
```

---

## 🐳 Docker Commands

### Development
```bash
# Build and start
docker-compose up --build

# Start in background
docker-compose up -d

# View logs
docker-compose logs -f api

# Stop services
docker-compose down
```

### Production with Monitoring
```bash
# Start with Prometheus + Grafana
docker-compose --profile monitoring up -d

# Check service health
docker-compose ps

# Execute commands in container
docker-compose exec api alembic upgrade head
```

### Cleanup
```bash
# Stop and remove volumes
docker-compose down -v

# Remove all containers and images
docker-compose down --rmi all
```

---

## 📊 Monitoring & Metrics

### Prometheus Metrics
Access metrics at: `http://localhost:8000/metrics`

**Available Metrics:**
- `http_requests_total` - Total HTTP requests
- `http_request_duration_seconds` - Request latency
- `http_requests_in_progress` - Active requests
- Custom application metrics

### Grafana Setup
1. Access: http://localhost:3001
2. Login: admin / admin (change on first login)
3. Add Prometheus data source: http://prometheus:9090
4. Import dashboard or create custom visualizations

---

## 🧪 Load Testing

### Run with Locust
```bash
# Install locust
pip install locust

# Start load test
locust -f locustfile.py --host=http://localhost:8000

# Access web UI
# Open http://localhost:8089
```

### Scenarios Tested
- User login flow
- Dashboard stats
- Issue/Return transactions
- ML analytics
- Vision scanning
- Admin operations

---

## 🔒 Security Hardening

### TLS/HTTPS Configuration
Use reverse proxy (Nginx/Traefik):

```nginx
server {
    listen 443 ssl http2;
    server_name api.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /ws/ {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Environment Variables
```bash
# Production .env
ENVIRONMENT=production
SECRET_KEY=<generate-strong-random-key>
SQLALCHEMY_DATABASE_URI=postgresql://user:pass@postgres:5432/library_db
REDIS_URL=redis://redis:6379/0
BACKEND_CORS_ORIGINS=["https://yourdomain.com"]
```

---

## 🔧 CI/CD Pipeline

### GitHub Actions
Workflow file: `.github/workflows/ci.yml`

**Pipeline Steps:**
1. ✅ Lint with flake8
2. ✅ Format check with black
3. ✅ Run tests with pytest
4. ✅ Security scan with Trivy
5. ✅ Build Docker image
6. ✅ Test Docker image

**Trigger:**
- Push to `main` or `develop`
- Pull requests

---

## 📈 Performance Tuning

### Database Indexing
All foreign keys and frequently queried fields are indexed:
- `transactions.user_id`
- `transactions.book_id`
- `books.isbn`
- `visit_logs.user_id`

### Gunicorn Workers
```bash
# Calculate optimal workers: (2 x CPU cores) + 1
workers = (2 x 4) + 1 = 9

# Update docker-compose.yml
CMD ["gunicorn", "app.main:app", "-k", "uvicorn.workers.UvicornWorker", "--workers", "9"]
```

### Redis Connection Pooling
Automatically handled by `aioredis` client.

---

## 🔍 Troubleshooting

### WebSocket Issues
```bash
# Check Redis connection
docker-compose exec redis redis-cli ping

# View WebSocket logs
docker-compose logs -f api | grep WebSocket
```

### Database Connection
```bash
# Test PostgreSQL
docker-compose exec postgres psql -U postgres -d library_db

# Run migration
docker-compose exec api alembic upgrade head
```

### Container Health
```bash
# Check all services
docker-compose ps

# Inspect container
docker-compose exec api python -c "from app.main import app; print('OK')"
```

---

## 📝 Maintenance

### Backups
```bash
# Database backup via API
curl -X POST http://localhost:8000/api/v1/admin/backup \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"

# Manual PostgreSQL backup
docker-compose exec postgres pg_dump -U postgres library_db > backup.sql

# Restore
docker-compose exec -T postgres psql -U postgres library_db < backup.sql
```

### Logs
```bash
# View application logs
docker-compose logs -f api

# Export logs
docker-compose logs api > application.log
```

### Updates
```bash
# Pull latest changes
git pull

# Rebuild and restart
docker-compose down
docker-compose up --build -d
```

---

## ✅ Production Checklist

- [ ] Update `.env` with production values
- [ ] Set `ENVIRONMENT=production`
- [ ] Configure strong `SECRET_KEY`
- [ ] Set up PostgreSQL with backups
- [ ] Configure Redis persistence
- [ ] Set proper `BACKEND_CORS_ORIGINS`
- [ ] Run migrations: `alembic upgrade head`
- [ ] Configure TLS/HTTPS (reverse proxy)
- [ ] Set up monitoring (Prometheus + Grafana)
- [ ] Configure log aggregation
- [ ] Test health endpoint: `/health`
- [ ] Test metrics endpoint: `/metrics`
- [ ] Run load tests
- [ ] Configure automated backups
- [ ] Set up CI/CD pipeline
- [ ] Document disaster recovery procedures

---

## 🎯 Next Steps

1. **Scale Horizontally**: Add more API workers
2. **Add CDN**: For static assets
3. **Implement Caching**: Redis for read-heavy endpoints
4. **Advanced Monitoring**: APM tools (New Relic, DataDog)
5. **Kubernetes**: For orchestration at scale
