# Twenty CRM Coolify Deployment Guide

## Prerequisites

- Coolify instance running and accessible
- Domain name pointed to your Coolify server
- At least 4GB RAM available on your server
- Git repository with your Twenty CRM code (or use the docker-compose directly)

## Important: Restore Coolify-Specific Configuration

Before deploying to Coolify, you need to restore one property that was removed for local testing:

### 1. Edit docker-compose.yaml

Add back the `exclude_from_hc: true` line to the `change-vol-ownership` service:

```yaml
change-vol-ownership:
  image: ubuntu
  user: root
  restart: "no"
  exclude_from_hc: true  # ← ADD THIS LINE BACK
  volumes:
    - server-local-data:/tmp/server-local-data
    - docker-data:/tmp/docker-data
```

This tells Coolify to exclude this initialization container from health checks.

## Deployment Steps

### Step 1: Create New Resource in Coolify

1. Log into your Coolify dashboard
2. Navigate to your project
3. Click **"+ New Resource"**
4. Select **"Docker Compose"**

### Step 2: Configure the Resource

#### Option A: From Git Repository

1. Select **"From Git"**
2. Connect your Git repository
3. Specify the path to `docker-compose.yaml`
4. Set branch (usually `main` or `master`)

#### Option B: Direct Docker Compose

1. Select **"Docker Compose"**
2. Paste the contents of your `docker-compose.yaml` file
3. Name your service (e.g., "twenty-crm")

### Step 3: Configure Environment Variables

Coolify will auto-populate these **magic variables** - you don't need to set them manually:

- ✅ `SERVICE_FQDN_SERVER` - Auto-populated with your domain
- ✅ `SERVICE_PASSWORD_POSTGRES` - Auto-generated secure password
- ✅ `SERVICE_BASE64_32_SECRET` - Auto-generated secret

**You DO need to set these variables manually:**

| Variable | Value | Required |
|----------|-------|----------|
| `TWENTY_VERSION` | `latest` or specific version | Optional (defaults to latest) |
| `DOMAIN` | Your domain (e.g., `crm.yourdomain.com`) | **Required** |
| `STORAGE_TYPE` | `local` | Optional (defaults to local) |

**Optional OAuth/Email variables** (only if you want these features):

<details>
<summary>Google OAuth (Click to expand)</summary>

```
MESSAGING_PROVIDER_GMAIL_ENABLED=true
CALENDAR_PROVIDER_GOOGLE_ENABLED=true
AUTH_GOOGLE_CLIENT_ID=your_client_id
AUTH_GOOGLE_CLIENT_SECRET=your_client_secret
AUTH_GOOGLE_CALLBACK_URL=https://your-domain.com/auth/google/callback
AUTH_GOOGLE_APIS_CALLBACK_URL=https://your-domain.com/auth/google-apis/callback
```
</details>

<details>
<summary>Microsoft OAuth (Click to expand)</summary>

```
CALENDAR_PROVIDER_MICROSOFT_ENABLED=true
MESSAGING_PROVIDER_MICROSOFT_ENABLED=true
AUTH_MICROSOFT_ENABLED=true
AUTH_MICROSOFT_CLIENT_ID=your_client_id
AUTH_MICROSOFT_CLIENT_SECRET=your_client_secret
AUTH_MICROSOFT_CALLBACK_URL=https://your-domain.com/auth/microsoft/callback
AUTH_MICROSOFT_APIS_CALLBACK_URL=https://your-domain.com/auth/microsoft-apis/callback
```
</details>

<details>
<summary>Email Configuration (Click to expand)</summary>

```
EMAIL_FROM_ADDRESS=noreply@yourdomain.com
EMAIL_FROM_NAME=Twenty CRM
EMAIL_SYSTEM_ADDRESS=system@yourdomain.com
EMAIL_DRIVER=smtp
EMAIL_SMTP_HOST=smtp.yourdomain.com
EMAIL_SMTP_PORT=465
EMAIL_SMTP_USER=your_smtp_user
EMAIL_SMTP_PASSWORD=your_smtp_password
```
</details>

### Step 4: Configure Domain

1. In Coolify, go to the **"Domains"** tab
2. Set your domain (e.g., `crm.yourdomain.com`)
3. Ensure your DNS A record points to your Coolify server IP
4. Coolify will automatically provision SSL via Let's Encrypt

### Step 5: Deploy

1. Click **"Deploy"** or **"Start"**
2. Coolify will:
   - Pull the Docker images
   - Create volumes
   - Start services in order (db → server → worker)
   - Configure Traefik reverse proxy
   - Provision SSL certificate

### Step 6: Monitor Deployment

1. Watch the **deployment logs** in Coolify dashboard
2. Check service health status
3. Wait for all services to show as "healthy" (typically 2-3 minutes)

### Step 7: Access Your CRM

Once deployed, navigate to your domain:
```
https://crm.yourdomain.com
```

You should see the Twenty CRM signup page.

## Troubleshooting Coolify Deployment

### Issue: Services Won't Start

**Check:**
1. **Logs**: View service logs in Coolify dashboard
2. **Magic Variables**: Verify they're populated in Environment tab
3. **Resources**: Ensure server has enough RAM (4GB minimum)

**Common causes:**
- Database not healthy before server starts
- Missing environment variables
- Insufficient server resources

### Issue: Database Connection Errors

**Symptoms:**
```
Error: could not connect to database
FATAL: password authentication failed
```

**Solution:**
1. Check if `SERVICE_PASSWORD_POSTGRES` is set in environment
2. Verify database container is healthy
3. Check database logs: Look for PostgreSQL startup errors
4. Ensure network connectivity between services

### Issue: Domain Not Accessible

**Symptoms:**
- Can't access via domain
- SSL certificate not provisioning
- 502 Bad Gateway

**Solution:**
1. **DNS**: Verify A record points to Coolify server IP
   ```bash
   nslookup crm.yourdomain.com
   ```
2. **Traefik**: Check Traefik labels in docker-compose
3. **SSL**: Wait 2-3 minutes for Let's Encrypt provisioning
4. **Service Health**: Ensure server service is healthy and listening on port 3000

### Issue: Server Starts But Shows Error Page

**Check:**
1. Server logs for specific errors
2. `APP_SECRET` is set (should be auto-populated)
3. `FRONTEND_URL` and `SERVER_URL` match your domain
4. Database migrations completed successfully

### Issue: Worker Service Failing

**Common causes:**
- Database not accessible
- Redis not accessible
- Missing environment variables

**Solution:**
1. Check worker logs
2. Verify `REDIS_URL` is correct (`redis://redis:6379`)
3. Ensure database is healthy
4. Check `DISABLE_DB_MIGRATIONS=true` is set (migrations run on server only)

## Verifying Deployment

### 1. Check Service Health

In Coolify dashboard, all services should show:
- ✅ `db` - Healthy
- ✅ `redis` - Healthy
- ✅ `server` - Healthy
- ✅ `worker` - Running
- ✅ `change-vol-ownership` - Completed

### 2. Test Database Connection

From Coolify terminal or SSH:
```bash
docker exec -it <db-container-name> pg_isready -U postgres
```

Should return: `accepting connections`

### 3. Test Redis Connection

```bash
docker exec -it <redis-container-name> redis-cli ping
```

Should return: `PONG`

### 4. Check Server Logs

Look for successful startup message:
```bash
docker logs <server-container-name>
```

Should see: Server listening on port 3000 or similar

### 5. Access Application

Navigate to your domain and verify:
- ✅ Page loads without errors
- ✅ SSL certificate is valid (green padlock)
- ✅ Can access signup page
- ✅ Can create account

## Environment Variables Reference

### Auto-Populated by Coolify (Don't Set Manually)

| Variable | Description |
|----------|-------------|
| `SERVICE_FQDN_SERVER` | Full domain URL for your Twenty instance |
| `SERVICE_PASSWORD_POSTGRES` | Auto-generated PostgreSQL password |
| `SERVICE_BASE64_32_SECRET` | Auto-generated application secret |

### Required Manual Configuration

| Variable | Example | Description |
|----------|---------|-------------|
| `DOMAIN` | `crm.yourdomain.com` | Your domain name |

### Optional Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `TWENTY_VERSION` | `latest` | Twenty CRM version to deploy |
| `STORAGE_TYPE` | `local` | Storage backend (local or s3) |
| `SIGN_IN_PREFILLED` | `true` | Pre-fill sign-in form |

## Post-Deployment Configuration

### Setting Up S3 Storage (Optional)

If you want to use S3 instead of local storage:

1. Add these environment variables in Coolify:
```
STORAGE_TYPE=s3
STORAGE_S3_REGION=us-east-1
STORAGE_S3_NAME=your-bucket-name
STORAGE_S3_ENDPOINT=https://s3.amazonaws.com
```

2. Redeploy the service

### Configuring OAuth (Optional)

See the expandable sections in Step 3 above for Google and Microsoft OAuth configuration.

### Setting Up Email (Optional)

Configure SMTP settings to enable email notifications and invitations.

## Maintenance

### Viewing Logs

In Coolify dashboard:
1. Navigate to your Twenty resource
2. Click on specific service (server, worker, db, redis)
3. View real-time logs

### Updating Twenty CRM

1. Change `TWENTY_VERSION` environment variable to desired version
2. Click **"Redeploy"** in Coolify
3. Coolify will pull new image and restart services

### Backing Up Data

Database and volumes are persisted in Docker volumes:
- `db-data` - PostgreSQL data
- `server-local-data` - Uploaded files
- `redis-data` - Cache data

Use Coolify's backup features or Docker volume backup tools.

### Restarting Services

In Coolify dashboard:
1. Navigate to service
2. Click **"Restart"**

Or restart all services:
1. Click **"Redeploy"** on the resource

## Quick Checklist

Before deploying to Coolify:

- [ ] Restore `exclude_from_hc: true` in docker-compose.yaml
- [ ] Set `DOMAIN` environment variable
- [ ] Configure DNS A record
- [ ] Verify server has 4GB+ RAM
- [ ] (Optional) Configure OAuth credentials
- [ ] (Optional) Configure SMTP settings

After deployment:

- [ ] Verify all services are healthy
- [ ] Access domain and check SSL
- [ ] Create first account
- [ ] Test core functionality
- [ ] Configure backup strategy
