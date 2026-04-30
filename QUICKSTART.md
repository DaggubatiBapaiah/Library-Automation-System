# 🚀 Quick Start Guide - Docker Deployment

## Prerequisites
- Docker Desktop installed
- Docker Compose installed
- 8GB RAM minimum
- 10GB free disk space

---

## ⚡ 5-Minute Deploy

### Step 1: Clone & Setup
```bash
git clone <repository-url>
cd freelancing-1
cp .env.example .env
```

### Step 2: Update Environment
Edit `.env`:
```bash
# Minimal required changes
SECRET_KEY=your-super-secret-key-here
ENVIRONMENT=production
```

### Step 3: Deploy
```bash
docker-compose up --build
```

### Step 4: Verify
```bash
# Check health
curl http://localhost:8000/health

# View API docs
# Open: http://localhost:8000/docs

# Check metrics
curl http://localhost:8000/metrics
```

---

## 🎯 What Gets Deployed

- **PostgreSQL** @ port 5432
- **Redis** @ port 6379
- **FastAPI API** @ port 8000
- **Automated Migrations** (Alembic)
- **Health Checks** (all services)
- **Prometheus Metrics** @ `/metrics`

---

## 🛠️ Common Commands

### Start Services
```bash
# Foreground (see logs)
docker-compose up

# Background (detached)
docker-compose up -d
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api
docker-compose logs -f postgres
docker-compose logs -f redis
```

### Stop Services
```bash
# Stop (keep data)
docker-compose down

# Stop + remove volumes (⚠️ deletes data!)
docker-compose down -v
```

### Database Operations
```bash
# Run migrations
docker-compose exec api alembic upgrade head

# Create new migration
docker-compose exec api alembic revision --autogenerate -m "Your message"

# Access PostgreSQL
docker-compose exec postgres psql -U postgres -d library_db
```

### Check Service Health
```bash
# All services status
docker-compose ps

# Individual health checks
docker-compose exec api curl http://localhost:8000/health
docker-compose exec redis redis-cli ping
docker-compose exec postgres pg_isready -U postgres
```

---

## 📊 Optional: Start with Monitoring

```bash
# Start with Prometheus + Grafana
docker-compose --profile monitoring up -d

# Access:
# - API: http://localhost:8000
# - Prometheus: http://localhost:9090
# - Grafana: http://localhost:3001 (admin/admin)
```

---

## 🧪 Load Testing

```bash
# Install locust
pip install locust

# Run load test
locust -f locustfile.py --host=http://localhost:8000

# Access web UI
# http://localhost:8089
```

---

## 🔒 Production Checklist

Before deploying to production:

- [ ] Change `SECRET_KEY` in `.env`
- [ ] Set `ENVIRONMENT=production`
- [ ] Configure proper `BACKEND_CORS_ORIGINS`
- [ ] Update database credentials
- [ ] Set up TLS/HTTPS (reverse proxy)
- [ ] Configure backups
- [ ] Set up monitoring alerts
- [ ] Review security settings

---

## 📚 Full Documentation

- **Comprehensive Guide**: `DEPLOYMENT.md`
- **Enterprise Features**: `ENTERPRISE_READINESS.md`
- **Production Checklist**: `DEPLOYMENT_CHECKLIST.md`
- **Stability Features**: `PRODUCTION_STABILITY.md`

---

## 🆘 Troubleshooting

### Container won't start
```bash
# Check logs
docker-compose logs api

# Rebuild from scratch
docker-compose down -v
docker-compose up --build
```

### Database connection issues
```bash
# Wait for PostgreSQL to be ready
docker-compose up -d postgres
# Wait 10 seconds
docker-compose up -d api
```

### Port conflicts
Edit `docker-compose.yml` ports section:
```yaml
ports:
  - "8001:8000"  # Change 8000 to 8001
```

---

## ✅ Success Indicators

You'll know it's working when:

1. ✅ All 3 services show "healthy" in `docker-compose ps`
2. ✅ Health endpoint returns: `{"status": "healthy"}`
3. ✅ API docs accessible at `/docs`
4. ✅ Metrics endpoint returns Prometheus data
5. ✅ No errors in logs

---

## 🎉 Next Steps

1. **Create Admin User**: Access `/docs` and use auth endpoints
2. **Test Features**: Try vision scanning, face recognition
3. **Configure Monitoring**: Set up Grafana dashboards
4. **Run Load Tests**: Baseline performance
5. **Set Up Backups**: Automated database dumps

---

**Questions?** Check `DEPLOYMENT.md` or raise an issue on GitHub.

**Ready for Production?** Review `ENTERPRISE_READINESS.md` checklist.
