# Twenty CRM Deployment Guide

## Prerequisites

Before deploying Twenty CRM, ensure you have:

- **Docker Desktop** installed and running
- **Docker Compose** (included with Docker Desktop)
- At least 4GB of available RAM
- At least 10GB of free disk space

## Local Deployment

### 1. Start Docker Desktop

Make sure Docker Desktop is running on your system. You can verify this by running:

```powershell
docker ps
```

If you see an error about connecting to the Docker API, start Docker Desktop and wait for it to fully initialize.

### 2. Environment Configuration

The `.env` file has been created with the following configuration:

- **Database Password**: `TwentyLocalDB2026SecurePass`
- **App Secret**: Auto-generated base64 secret
- **Frontend URL**: `http://localhost:3000`
- **Storage**: Local file storage

You can modify these values in the `.env` file if needed.

### 3. Start the Services

```powershell
docker-compose up -d
```

This will start the following services:
- **PostgreSQL** database (port 5432)
- **Redis** cache (port 6379)
- **Twenty Server** (port 3000)
- **Twenty Worker** (background jobs)

### 4. Monitor the Startup

Watch the logs to ensure everything starts correctly:

```powershell
docker-compose logs -f server
```

Wait for the message indicating the server is ready (usually shows "Server started" or similar).

### 5. Access Twenty CRM

Once the server is ready, open your browser and navigate to:

```
http://localhost:3000
```

You should see the Twenty CRM login/signup page.

### 6. Create Your First Account

Click "Sign Up" and create your first workspace account.

## Troubleshooting

### Docker Desktop Not Running

**Error**: `failed to connect to the docker API`

**Solution**: Start Docker Desktop and wait for it to fully initialize before running docker-compose commands.

### Port Already in Use

**Error**: `port is already allocated`

**Solution**: Either stop the service using that port or modify the port mapping in `docker-compose.yaml`.

### Database Connection Issues

**Error**: `could not connect to database`

**Solution**:
1. Check if the database container is healthy: `docker-compose ps`
2. View database logs: `docker-compose logs db`
3. Ensure the password in `.env` matches the docker-compose configuration

### Server Won't Start

**Solution**:
1. Check server logs: `docker-compose logs server`
2. Verify all environment variables are set correctly in `.env`
3. Ensure the database is healthy before the server starts

## Useful Commands

### View All Container Status
```powershell
docker-compose ps
```

### View Logs for All Services
```powershell
docker-compose logs -f
```

### View Logs for Specific Service
```powershell
docker-compose logs -f server
docker-compose logs -f db
docker-compose logs -f redis
```

### Restart Services
```powershell
docker-compose restart
```

### Stop Services
```powershell
docker-compose down
```

### Stop Services and Remove Volumes (Clean Start)
```powershell
docker-compose down -v
```

### Rebuild and Restart
```powershell
docker-compose up -d --build
```

---

## Coolify Deployment

### Understanding Coolify Magic Variables

The `docker-compose.yaml` file uses Coolify-specific "magic variables" that are automatically populated by Coolify:

| Variable | Coolify Value | Local Value |
|----------|---------------|-------------|
| `SERVICE_FQDN_SERVER` | Your domain URL | `http://localhost:3000` |
| `SERVICE_PASSWORD_POSTGRES` | Auto-generated password | Manual password in `.env` |
| `SERVICE_BASE64_32_SECRET` | Auto-generated secret | Manual secret in `.env` |
| `DOMAIN` | Your domain | `localhost` |

### Coolify-Specific Properties

The original docker-compose file contained `exclude_from_hc: true` which is a Coolify-specific property. This has been removed for local deployment but should be present for Coolify deployments.

### Deploying to Coolify

1. **Create a New Resource** in Coolify
   - Select "Docker Compose" as the resource type
   - Upload or paste your `docker-compose.yaml`

2. **Configure Environment Variables** in Coolify UI
   - Coolify will auto-populate `SERVICE_*` variables
   - Add any optional variables (OAuth, Email, etc.)

3. **Deploy**
   - Coolify will handle the deployment automatically
   - Monitor logs in the Coolify dashboard

### Common Coolify Issues

#### Services Not Starting

**Check**:
1. Coolify logs in the dashboard
2. Verify all magic variables are being populated
3. Ensure your server has enough resources

#### Database Connection Failures

**Check**:
1. Verify `SERVICE_PASSWORD_POSTGRES` is set
2. Check if database service is healthy
3. Review database logs in Coolify

#### Domain Not Accessible

**Check**:
1. DNS records are correctly configured
2. SSL certificate is provisioned
3. Traefik labels are correct in docker-compose

### Differences Between Local and Coolify Deployment

| Aspect | Local | Coolify |
|--------|-------|---------|
| Environment Variables | Manual `.env` file | Auto-populated magic variables |
| Domain | localhost | Your custom domain |
| SSL | Not configured | Automatic via Let's Encrypt |
| Reverse Proxy | Direct access | Traefik |
| Secrets | Manual generation | Auto-generated |
| Health Checks | Standard Docker | Enhanced with Coolify monitoring |

### Restoring Coolify-Specific Properties

If deploying to Coolify, restore this line to the `change-vol-ownership` service:

```yaml
exclude_from_hc: true
```

This tells Coolify to exclude this service from health checks since it's a one-time initialization container.

## Next Steps

After successful local deployment:

1. ✅ Test core CRM functionality
2. ✅ Create test data (contacts, companies, etc.)
3. ✅ Verify data persistence (restart containers)
4. ✅ Configure optional features (OAuth, Email)
5. ✅ Deploy to Coolify using the same configuration
