# n8n Docker Compose Documentation

## Overview
This documentation covers the n8n workflow automation platform deployed using Docker Compose. n8n is an extendable workflow automation tool that connects different services and automates tasks without coding.

---

## Architecture

### Service: n8n
- **Image**: `n8nio/n8n` (official n8n image from Docker Hub)
- **Purpose**: Workflow automation and integration platform
- **Port**: `5678` (web interface)
- **Volume**: `./n8n_data` → `/home/node/.n8n` (persistent data storage)

---

## Configuration Details

### Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| `TZ` | `Asia/Kathmandu` | Timezone for scheduled workflows and logs |
| `N8N_BASIC_AUTH_ACTIVE` | `true` | Enables basic authentication |
| `N8N_BASIC_AUTH_USER` | `admin` | Default login username |
| `N8N_BASIC_AUTH_PASSWORD` | `admin123` | Default login password |

### Ports

| Port | Mapping | Purpose |
|------|---------|---------|
| `5678` | Container → Host | n8n web interface (http://localhost:5678) |

### Volumes

| Host Path | Container Path | Purpose |
|-----------|-----------------|---------|
| `./n8n_data` | `/home/node/.n8n` | Persistent storage for workflows, credentials, and execution history |

---

## Quick Start

### Prerequisites
- Docker Engine (20.10+)
- Docker Compose (1.29+)
- Minimum 512MB available disk space

### Starting the Service

```bash
docker-compose up -d
```

**Output:**
```
Creating n8n ... done
```

### Access n8n
- **URL**: http://localhost:5678
- **Username**: `admin`
- **Password**: `admin123`

### Stopping the Service

```bash
docker-compose down
```

### Viewing Logs

```bash
docker-compose logs -f n8n
```

---

## Data Persistence

All n8n data is stored in `./n8n_data/` directory:
- Workflow definitions
- Credentials and secrets
- Execution history
- User settings

**Backup:**
```bash
cp -r ./n8n_data ./n8n_data_backup_$(date +%Y%m%d)
```

**Restore:**
```bash
rm -rf ./n8n_data
cp -r ./n8n_data_backup_20240101 ./n8n_data
docker-compose up -d
```

---

## Security Considerations

### ⚠️ Production Warnings

1. **Weak Credentials**: Change default password immediately
   ```bash
   # In docker-compose.yml
   N8N_BASIC_AUTH_PASSWORD=your_secure_password_here
   ```

2. **Authentication**: Consider using OAuth2 or LDAP for production
   ```yaml
   N8N_OAUTH_CLIENT_ID=your_client_id
   N8N_OAUTH_CLIENT_SECRET=your_client_secret
   ```

3. **Port Exposure**: Use a reverse proxy (nginx, Traefik) in production
   - Do NOT expose port 5678 directly to the internet
   - Use HTTPS/TLS encryption

4. **Secrets Management**: Store sensitive credentials in `.env` file
   ```bash
   # .env
   N8N_BASIC_AUTH_PASSWORD=${SECURE_PASSWORD}
   ```

5. **Network**: Create isolated Docker network for multi-service setups
   ```yaml
   networks:
     backend:
       driver: bridge
   ```

---

## Advanced Configuration

### Environment Variables Reference

```yaml
environment:
  - TZ=Asia/Kathmandu
  - N8N_BASIC_AUTH_ACTIVE=true
  - N8N_BASIC_AUTH_USER=admin
  - N8N_BASIC_AUTH_PASSWORD=admin123
  # Optional additions:
  - N8N_HOST=0.0.0.0
  - N8N_PORT=5678
  - N8N_PROTOCOL=http
  - N8N_LOG_LEVEL=info
  - WEBHOOK_URL=http://localhost:5678
  - DATABASE_TYPE=sqlite
```

### Using PostgreSQL Database (Recommended for Production)

```yaml
version: "3"

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: secure_password
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - TZ=Asia/Kathmandu
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=admin123
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=secure_password
      - DB_POSTGRESDB_DATABASE=n8n
    volumes:
      - ./n8n_data:/home/node/.n8n
    depends_on:
      - postgres

volumes:
  postgres_data:
```

### Memory and CPU Limits

```yaml
n8n:
  image: n8nio/n8n
  # ... other config ...
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 1G
      reservations:
        cpus: '1'
        memory: 512M
```

---

## Troubleshooting

### Container fails to start
```bash
docker-compose logs n8n
```
**Common causes:**
- Port 5678 already in use: `lsof -i :5678` or change `ports: ["6789:5678"]`
- Insufficient permissions on `./n8n_data`: `chmod 755 n8n_data`
- Corrupted SQLite database: Remove `./n8n_data/database.sqlite` and restart

### Workflows not executing
- Check logs: `docker-compose logs n8n`
- Verify webhook URL is reachable: `curl -I http://localhost:5678`
- Check n8n event logs in UI

### Slow performance
- Increase memory limit (see Advanced Configuration)
- Switch to PostgreSQL database
- Archive old execution history in n8n UI

### Lost credentials after restart
- Verify `./n8n_data` volume is mounted and persistent
- Check `docker inspect n8n` for volume mounts
- Restore from backup if needed

---

## Maintenance

### Updates

Check for new versions:
```bash
docker pull n8nio/n8n:latest
```

Update and restart:
```bash
docker-compose down
# Update image tag in docker-compose.yml if needed
docker-compose up -d
```

### Cleanup

Remove stopped containers and unused images:
```bash
docker system prune -a
```

Remove n8n data (WARNING: irreversible):
```bash
docker-compose down -v
```

---

## Common Workflows

### Import Workflow
1. In n8n UI: **Workflow** → **Import from File**
2. Select `.json` file exported from another n8n instance

### Export Workflow
1. In n8n UI: **Workflow** → **Download**
2. Saves as `.json` file

### Add Integrations
1. In n8n UI: **Credentials** → **New**
2. Select service (Slack, GitHub, etc.)
3. Enter API keys and authenticate

---

## Performance Metrics

Monitor resource usage:
```bash
docker stats n8n
```

Expected baseline (idle):
- **Memory**: 150-250MB
- **CPU**: <1%

---

## Support & Resources

- **Official Documentation**: https://docs.n8n.io/
- **Docker Hub**: https://hub.docker.com/r/n8nio/n8n
- **Community Forum**: https://community.n8n.io/
- **GitHub Issues**: https://github.com/n8n-io/n8n/issues

---

## Checklist: Before Going to Production

- [ ] Change `N8N_BASIC_AUTH_PASSWORD` to a strong password
- [ ] Set `TZ` to your actual timezone
- [ ] Use PostgreSQL or external database (not SQLite)
- [ ] Enable HTTPS/TLS with reverse proxy
- [ ] Set up regular backups of `./n8n_data`
- [ ] Configure resource limits (`deploy.resources`)
- [ ] Enable logging and monitoring
- [ ] Test disaster recovery procedures
- [ ] Document all custom workflows and integrations
- [ ] Set up automated updates or a maintenance window

---

**Version**: 1.0  
**Last Updated**: 2024  
**Docker Compose Version**: 3
