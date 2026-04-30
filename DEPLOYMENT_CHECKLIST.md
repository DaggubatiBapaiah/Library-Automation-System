# Production Deployment Checklist

## Pre-Deployment Configuration

### Environment Variables (.env)
```bash
# Copy example and update
cp .env.example .env

# Required Updates:
- ENVIRONMENT=production
- SECRET_KEY=<generate-strong-random-key>
- SQLALCHEMY_DATABASE_URI=postgresql://user:pass@host/db  # Use PostgreSQL in production
- BACKEND_CORS_ORIGINS=https://yourdomain.com,https://app.yourdomain.com
- ACCESS_TOKEN_EXPIRE_MINUTES=11520  # Or adjust as needed
```

### Database Setup
```bash
# For PostgreSQL (recommended for production)
# 1. Create database
createdb library_db

# 2. Update .env with connection string
SQLALCHEMY_DATABASE_URI=postgresql://user:password@localhost/library_db

# 3. Initialize
python -m app.initial_data
```

### System Requirements
- [ ] Python 3.9+
- [ ] Tesseract OCR installed (for vision module)
- [ ] Sufficient disk space for backups and models
- [ ] PostgreSQL installed (recommended)

## Security Checklist

- [ ] Changed default SECRET_KEY
- [ ] Updated FIRST_SUPERUSER credentials in .env
- [ ] Verified CORS origins match your frontend domain
- [ ] ENVIRONMENT set to "production"
- [ ] Database credentials are secure
- [ ] Admin endpoints tested (should reject non-superusers)
- [ ] Rate limiting verified on face verification
- [ ] SSL/TLS configured (reverse proxy level)

## Performance Checklist

- [ ] Database indexes verified:
  ```sql
  -- Check indexes exist
  SELECT tablename, indexname FROM pg_indexes WHERE schemaname = 'public';
  ```
- [ ] Number of Gunicorn workers = (2 x CPU cores) + 1
- [ ] WebSocket connections monitored
- [ ] Background task loop running (check logs)

## Functionality Verification

### Core Features
- [ ] JWT Login works
- [ ] Face verification works (after model download)
- [ ] Book CRUD operations
- [ ] Issue/Return transactions
- [ ] Shelf management

### Advanced Features
- [ ] Vision book scanning
- [ ] Automated return box
- [ ] ML analytics endpoints
- [ ] WebSocket dashboard updates
- [ ] Notification system
- [ ] Visit tracking

### Admin Features
- [ ] Database backup creates file
- [ ] Database restore works
- [ ] Only accessible to superusers

## Deployment Steps

### Option 1: Gunicorn (Recommended)
```bash
# Install gunicorn if not already
pip install gunicorn

# Run with 4 workers
gunicorn app.main:app \
  -k uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --workers 4 \
  --access-logfile - \
  --error-logfile -
```

### Option 2: Uvicorn
```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### Option 3: Docker (if containerized)
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "app.main:app", "-k", "uvicorn.workers.UvicornWorker", "--bind", "0.0.0.0:8000"]
```

## Reverse Proxy (Nginx Example)

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /ws/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## Post-Deployment Monitoring

### Logs
```bash
# View logs (if using systemd)
journalctl -u library-api -f

# Important patterns to watch:
grep "ERROR" app.log
grep "WARNING" app.log
grep "Face verification" app.log
grep "Admin" app.log
```

### Health Checks
```bash
# Verify API is up
curl http://localhost:8000/health

# Expected response:
# {"status": "healthy", "environment": "production", "version": "1.0.0"}
```

### Monitoring Metrics
- Response times (should be <200ms for most endpoints)
- Face verification success rate
- WebSocket active connections
- Database query times
- Overdue notification generation count

## Troubleshooting

### DeepFace Model Download Hangs
- First face registration triggers ~500MB download
- Ensure internet connectivity
- Pre-download: `python -c "from deepface import DeepFace; DeepFace.build_model('VGG-Face')"`

### Database Locks (SQLite)
- Migrate to PostgreSQL for production
- SQLite doesn't handle concurrent writes well

### WebSocket Disconnects
- Check CORS settings
- Verify reverse proxy WebSocket configuration
- Monitor connection count for leaks

### Rate Limiting False Positives
- Currently uses IP-based rate limiting
- Behind reverse proxy: ensure `X-Forwarded-For` is passed
- Consider migrating to Redis-based rate limiting

## Backup Strategy

### Automated Backups
```bash
# Cron job example (daily at 2 AM)
0 2 * * * curl -X POST http://localhost:8000/api/v1/admin/backup \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

### Manual Backup
```bash
# Via API
curl -X POST http://localhost:8000/api/v1/admin/backup \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"

# Direct database backup (PostgreSQL)
pg_dump library_db > backup_$(date +%Y%m%d).sql
```

## Performance Tuning

### Database
- Regular VACUUM (PostgreSQL)
- Monitor query performance
- Add indexes as needed

### Application
- Monitor memory usage (DeepFace models are heavy)
- Adjust worker count based on load
- Consider caching for ML analytics

## Final Verification

Run the test suite:
```bash
python test_production.py
```

Check all items in `PRODUCTION_STABILITY.md`.

## Go-Live Decision

Only proceed when ALL items above are checked and verified.

**Status**: ☐ Ready for Production
**Deployed By**: _______________
**Date**: _______________
**Reviewed By**: _______________
